---
language:
- en
license: proprietary
tags:
- technical-showcase
- true-moe
- sparse-activation
- 4-bit-nf4
- nvidia-gtx-1060
- custom-kernel
pipeline_tag: text-generation
inference: false
---

# PROJECT LIA: 22B Effective Parameter True-MoE Engine on 6GB VRAM
> **LIA — Large Intelligent Agent**  
> **TECHNICAL SHOWCASE / PROPRIETARY RESEARCH**
>
> *This repository is a read-only artifact gallery demonstrating the feasibility of running datacenter-class Mixture-of-Experts (MoE) models on legacy consumer hardware. Source code is closed-source.*

## The Engineering Challenge

### 1. The Physics Barrier (Hardware)

According to the official model card published by OpenAI, gpt-oss-20b is designed to run within 16GB of GPU memory using MXFP4 quantization. This is the recommended minimum configuration for inference of this model.

It is well-known in the local LLM community that this requirement can be circumvented through CPU/GPU hybrid offload strategies — these practices exist and are not novel. However, conventional hybrid offload typically recommends 32-64GB of fast DDR5 system RAM to compensate for the bandwidth bottleneck between CPU memory and GPU compute.

PROJECT LIA runs gpt-oss-20b on a configuration significantly below both thresholds, using a tri-tier memory hierarchy with active use of all three tiers:

GPU (Tier 1): NVIDIA GTX 1060 with 6GB VRAM — 2.6x less than the official 16GB minimum

System RAM (Tier 2): 24GB DDR3 at 2400 MHz — below the 32-64GB DDR5 recommendation, with memory generations behind in both capacity and bandwidth

SSD (Tier 3): Used as active extension of address space through Windows page file, holding 40+ GB of committed memory that supplements the limited physical RAM

The pipeline orchestrates data movement across all three tiers simultaneously: active computation in VRAM, recently-used components and routing decisions in RAM, and inactive expert weights on SSD. Placement decisions are driven by routing signals from the MoE layer rather than uniform layer distribution.

Operating evidence: during inference the system commits 63GB of virtual memory against 24GB physical RAM (40+ GB held in page file on SSD), while remaining stable and supporting concurrent workloads such as video streaming. This indicates the orchestration successfully prevents the SSD tier from blocking inference, despite SSD bandwidth being orders of magnitude lower than RAM or VRAM.
Hardware platform: GTX 1060 (consumer GPU released in 2016) on a laptop chassis with DDR3 RAM. The system operates below its operational ceiling rather than at its limit.

This is achieved through: per-module mixed-precision quantization (BF16/NF4) applied selectively across model components, manual placement of model elements across the three memory tiers based on access patterns and MoE routing, and adapter binding strategy that preserves LoRA functionality on the modified quantization profile.

Source: https://huggingface.co/openai/gpt-oss-20b

### 2. The Tooling Gap (Software)

At the time of development, default configurations from the HuggingFace ecosystem (transformers, accelerate, peft, bitsandbytes) failed at initialization for this specific combination of model and hardware. Standard device_map="auto" distribution, default offload configurations, and out-of-the-box quantization profiles consistently produced OOM errors or unstable inference on the 6GB VRAM / 24GB DDR3 RAM target.

The standard libraries provide the foundational building blocks, but for this configuration extensive manual work was required:

-Model loading: Custom device map definitions that distribute model components across VRAM, RAM, and SSD based on access patterns rather than uniform layer distribution.

-Quantization: Per-module mixed-precision profile (BF16/NF4) selected based on sensitivity analysis of each component, rather than uniform quantization across the model.

-Adapter binding: After non-standard quantization changes the internal layer structure, LoRA adapters cannot attach through standard PEFT target_modules logic. Manual reconstruction of attachment points was required to preserve adapter functionality on the modified base.

-Expert orchestration: MoE-aware logic for selective expert loading and placement, leveraging routing signals to predict which experts will be needed and minimize SSD access during inference.

-Tokenization and runtime coordination: Custom orchestration layer (~530 lines of Python) that manages model lifecycle, memory transitions, and inference flow control.

The resulting runtime is tightly coupled to this specific hardware/model combination — the parameter values, loading sequence, and placement strategy are not portable through standard tooling. This represents engineering work in the gap between "the library supports this in theory" and "this actually runs stably on consumer hardware below official minimum requirements."

## The Architecture

LIA combines a fine-tuned base model with a custom inference runtime. The system was built in two phases: training an extensive PEFT adapter on the base model to shape behavior and response patterns, then compressing all components to fit constrained hardware through a custom orchestration pipeline.

### 1. Architectural Expansion (20B → 22B)1. Base Model and Adapter Training (Phase 1)

The foundation is gpt-oss-20b — a 21B parameter MoE model with 3.6B active parameters per token, released by OpenAI under Apache 2.0 license.

An extensive PEFT adapter with 1,173,057,792 trainable parameters (5.3% of total parameter count) was trained using LoRA with bias="all" configuration. This setup trains not only the low-rank decomposition matrices but also all bias parameters throughout the model, producing a significantly larger and more impactful adapter than standard LoRA configurations.

The parameter count growth comes from behavioral logic, not from added knowledge. The training dataset (556 examples) consisted of custom instructions and behavioral protocols rather than factual content. The 9.08 GB adapter size reflects what the model learned about how to behave, respond, and process instructions — not new things it learned to know. 

This distinction matters: factual knowledge resides in the base model and remained unchanged; the adapter encodes the behavioral and procedural layer learned during training.

The artifact: A 9.08 GB checkpoint (adapter_model.bin), notably larger than typical LoRA adapters (which are usually 100MB-1GB for models of this class) due to the bias="all" configuration combined with extensive behavioral training.

Function: The base model retains its knowledge and reasoning capabilities; the adapter shapes how those capabilities are expressed through learned behavioral patterns and modified bias terms throughout the network.

Implementation details: Specific training hyperparameters (LoRA rank, learning rate, training duration, dataset composition) are proprietary.

### 2. Custom Compression (Phase 2)

To run the combined system on a GTX 1060 with 6GB VRAM, both the base model and the adapter were compressed using a custom mixed-precision quantization profile:

Base model: The 38.8 GB base weights were compressed to 11.0 GB using per-module mixed-precision quantization (BF16/NF4). The quantization format for each module was selected based on sensitivity analysis rather than applying uniform quantization across the entire model.

Adapter: The 9.08 GB adapter checkpoint was optimized to a 4.58 GB runtime form compatible with the compressed base.

Adapter rebinding: After non-standard quantization changes the internal layer structure, LoRA adapters cannot attach through standard PEFT target_modules logic. Manual reconstruction of attachment points was required to preserve adapter functionality on the modified base.

Implementation details: Specific per-module quantization mapping and adapter rebinding logic are proprietary.

### 3. Custom MoE Offload Pipeline

The runtime treats the GPU as a compute core rather than a storage unit, with model components distributed across a tri-tier memory hierarchy (VRAM / RAM / SSD) and orchestrated based on MoE routing decisions.

Orchestration layer: A custom userland Python runtime (~530 lines) manages model lifecycle, memory transitions, expert placement, and inference flow control.

Routing logic: Expert selection and prefetching are driven by MoE routing signals, with placement decisions optimized to minimize data movement across memory tiers during inference.

Tri-tier coordination: Active computation in VRAM, recently-used components in RAM, inactive expert weights and overflow on SSD (held in Windows page file). The orchestration prevents the SSD tier from blocking inference despite its significantly lower bandwidth.

Environment coupling: The runtime is tightly coupled to this specific hardware/model combination. Parameter values, loading sequence, and placement strategy are not portable through standard tooling.

Implementation details: Specific orchestration logic, prefetching strategies, and configuration parameters are proprietary.
---

## 📊 Performance Matrix: The Transformation

This table demonstrates the "Surgical" process: converting a massive unrunnable training checkpoint into a functioning runtime engine on consumer hardware.

| Metric | Original Training State (Source) | LIA Runtime Engine (Result) |
| :--- | :--- | :--- |
| **Base Architecture** | **GPT-OSS 20B (MoE)** | **Surgical MoE (Preserved)** |
| **Expansion Logic** | +2B Parameters (Training Phase) | Integrated Logic Stream |
| **Total Parameters** | **~22 Billion** (MoE) | **~22 Billion** (MoE) |
| **Base Weights (Knowledge)**| 38.8 GB (FP16 .bin) | **11.0 GB** (Surgical NF4) |
| **Logic Adapter (FSM)** | **9.08 GB** (Raw Training Artifact) | **4.58 GB** (Optimized Stream) |
| **VRAM Required** | 45GB+ (Full Load) | **4.6 GB** (Dynamic Swapping) |
| **State** | Uncompressed / Static | **Compressed / Fluid** |

---

## LIA includes several runtime Runtime Capabilities

LIA includes several runtime mechanisms beyond standard next-token prediction:

1. Dynamic Adapter Binding

Custom protocol that reconstructs adapter attachment points during model initialization, handling both the modified internal structure and the distributed memory layout. Implementation details are proprietary.

2. Hardware Orchestration Layer

Custom local control protocol coordinating compute and memory across the system. Required adjusting Windows system-level settings that are restricted by default. The runtime will not function on a standard "out-of-the-box" Windows configuration.

3. Fully Offline Operation

No external dependencies or network access. All model state and orchestration reside on local storage and memory.

---

## Visual Proofs & Artifacts

| Category | Artifact | Description |
| :--- | :--- | :--- |
| **The "Surgery"** | ![Original vs Rebuild](assets/Original_Base.jpg) <br> ![Rebuild](assets/REBuild_Base.jpg) | **Transformation:** 38.8GB Original weights (top) compressed into the 11GB Surgical Block (bottom). |
| **Logic Core** | ![Adapter Structure](assets/model_structure.jpg) | **The "Brain":** The massive 9GB `.bin` file (raw logic) alongside the optimized 4.5GB runtime stream. |
| **Memory Matrix** | ![Memory Usage](assets/memory_consumption.jpg) | **Zero-Crash Swapping:** Managing a **63GB Commit Charge** on consumer RAM/SSD without triggering OOM. |
| **Stability** | ![Performance](assets/Performance%20optimization.jpg) | **Multitasking:** Concurrent 22B inference + Video Streaming. Note the GPU is cold (~47°C) due to sparse activation. |
| **Cognition** | ![Reflection Log](assets/reflection_log.jpg) | **Internal Monologue:** The model analyzing intent ("Owner wants to be in charge") before responding. |

---

> **Status:** `OPERATIONAL / STABLE`
> **Developer:** Denys
> **Environment:** Localhost / Custom orchestration runtime on Windows

---


## Lineage & Acknowledgements

This architecture traces its genetic lineage to the **GPT-OSS-20B** foundation.
However, due to the surgical 20B→22B expansion, a custom NF4 quantization map,
and a fundamental shift from conventional fine-tuning to runtime systems engineering,
the resulting artifacts constitute a distinct evolutionary branch.

**LIA** is binary-incompatible with its ancestor and exists as a standalone
Cognitive Engine. Its operation depends on a specific "Host-as-Code" execution
environment, without which the model artifacts are non-functional.
