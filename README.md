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

## The "Impossible" Architecture

LIA is a result of a specific **evolutionary process**. We did not simply quantize a standard model. We grew a new neural structure (Logic Turbine) on top of a 20B base, and then surgically compressed the body to fit the chassis.

### 1. Architectural Expansion (20B → 22B)
The project started with a standard **20B MoE Base**. During the high-precision training phase, we cultivated a massive **Adapter Layer (+2B Parameters)**.
* **The Artifact:** This resulted in a **9.08 GB** binary file (`adapter_model.bin`) containing the pure FSM logic, Identity, and Protocol definitions formed *before* any compression took place.
* **Purpose:** Unlike "Knowledge" (stored in the base), this layer handles "Reasoning" and "Control".

### 2. Surgical Compression (The "Fit" Phase)
To run this 22B monster on a **GTX 1060 (6GB)**, we performed a split optimization:
* **Base Model (Knowledge):** The 38.8 GB base weights were aggressively compressed into an **11 GB Surgical Block** (NF4) to reside in system RAM.
* **Logic Stream (Intelligence):** The 9GB Adapter was optimized into a **4.58 GB Runtime Stream**.

### 3. The "Adaptive MoE Offload" Pipeline
The system bypasses the VRAM limit by treating the GPU as a "Compute Core" rather than a "Storage Unit".
* **Orchestration:** The Grace Hopper pipeline operates inside a custom Windows execution environment with a bespoke runtime kernel responsible for expert orchestration.
* **Routing Logic:** Expert selection and prefetching are driven by routing signals, tokenizer state, and manually wired execution paths.
* **Host Dependency:** This environment bypasses standard ML framework loaders and relies on a custom library stack and execution order, making the runtime non-reproducible outside its native system configuration.

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

## Runtime Capabilities

LIA includes several runtime mechanisms that go beyond standard next-token prediction:

1. **Tensor Integrity Checks**
   Detects and mitigates inactive or malformed expert tensors during dynamic swapping.

2. **Hardware-Aware Control Layer**
   Interfaces with host hardware through a restricted local control protocol.

3. **Fully Offline Operation**
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
> **Environment:** Localhost / Custom Windows Kernel

---


## Lineage & Acknowledgements

This architecture traces its genetic lineage to the **GPT-OSS-20B** foundation.
However, due to the surgical 20B→22B expansion, a custom NF4 quantization map,
and a fundamental shift from conventional fine-tuning to runtime systems engineering,
the resulting artifacts constitute a distinct evolutionary branch.

**LIA** is binary-incompatible with its ancestor and exists as a standalone
Cognitive Engine. Its operation depends on a specific "Host-as-Code" execution
environment, without which the model artifacts are non-functional.
