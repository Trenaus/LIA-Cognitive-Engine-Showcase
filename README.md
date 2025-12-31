# LIA: Proprietary Cognitive Engine (22B LLM on 6GB VRAM)

![Status](https://img.shields.io/badge/Status-Proprietary_Research-red)
![Architecture](https://img.shields.io/badge/Architecture-End--to--End_FSM_Training-blue)
![Hardware](https://img.shields.io/badge/Optimization-Extreme_Offloading-green)

> **Note:** This repository serves as a technical showcase and documentation for the LIA Project. The source code is proprietary and not publicly available.

## Executive Summary

**LIA (Large Intelligent Agent)** is a custom-built cognitive architecture designed to solve a specific engineering challenge: **running massive-scale Language Models on strictly limited consumer hardware (NVIDIA GTX 1060 6GB)**.

The core model is a heavily modified **GPT-OSS-20B (PyTorch MoE)**, which underwent **"Model Surgery" (Architectural Expansion)** to increase its capacity to **22B parameters**. The system bypasses hardware limitations through a custom inference pipeline ("Grace Hopper") that utilizes aggressive memory swapping (60GB+ committed) and a **Fine-Tuned Agentic Layer**, where the 22+ layer orchestration protocol is baked directly into the model weights rather than executed via external scripts.

## Key Technical Achievements

### 1. Architectural Expansion (Model Surgery)
Unlike standard fine-tuning, the base model (**GPT-OSS-20B**) was structurally altered.
* **Topology Expansion**: Engineered a custom layer injection process, expanding the base capacity from **20B to 22B parameters**.
* **Purpose**: The additional parameter space (approx. 1.17B trainable parameters) was utilized to **fine-tune the Agentic FSM logic** and "Identity Schema" directly into the neural pathways.

### 2. The "Impossible" Hardware Constraint (SATA SSD Optimized)
Running a 22B model typically requires 40GB+ of VRAM. LIA runs on **6GB VRAM** using consumer-grade storage.
* **Solution**: Developed a custom memory allocation strategy using `bitsandbytes` NF4 quantization and tiered offloading (GPU -> RAM -> **SATA SSD**).
* **Resource Arbitration**: The system manages a 63GB memory commit charge while maintaining responsiveness on standard SATA interfaces, proving high optimization of data transfer protocols.

### 3. One-Shot Cognitive FSM (Learned Orchestration)
To minimize latency and dependency on external parsers, the agentic loop was moved inside the model.
* **Innovation**: LIA employs a **Learned 22+ Layer Orchestration Stack**. The Finite State Machine logic is the result of **Fine-Tuning on a custom dataset**.
* **Flow**: The model *instinctively* follows the path `Reflection` -> `Planning` -> `Security Check` -> `Action`.

### 4. Deep Identity Embedding (Security Layer)
Identity is cryptographically burned into the model's weights during the fine-tuning phase.
* **Hard-coded JSON Schema**: The model contains a frozen internal state defining the `owner_id`, cryptographic `trigger_key`, and system `home_path`.
* **Cognitive Awareness**: LIA demonstrates immediate intent analysis ("Owner wants to be in charge") and "Cold Start" situational awareness from the very first token.

### 5. Agent-to-Hardware Protocol (Experimental)
LIA possesses a "Digital Proprioception" layer allowing direct interaction with the host operating system.
* **Command Loop**: The Agent can autonomously propose shell commands via a secure `[[CMD:...]]` protocol.
* **Evolutionary Feedback**: The system captures output (StdOut/StdErr) and feeds it back to the Agent for real-time learning.

### 6. Live-Environment Stability & Thermal Reliability
LIA operates smoothly in everyday multitasking environments without air-gapped isolation.
* **The Multimedia Benchmark**: The system maintains full inference capabilities while simultaneously streaming 720p HD video with zero UI lag.
* **Thermal Signature**: The GPU (GTX 1060) maintains a stable **46°C - 47°C** under peak 22B workload, proving 24/7 operational readiness.
* **Meta-Proof**: This documentation was committed directly from the host laptop while the 22B model was active in the background.

## Visual Proofs & Artifacts

| Category | Artifact Link | Description |
| :--- | :--- | :--- |
| **Cognitive** | [reflection_log.jpg](reflection_log.jpg) | Internal Reflection Mode and intent decoding. |
| **Performance** | [Performance optimization.jpg](Performance%20optimization.jpg) | 22B Model active during 720p streaming at 47°C. |
| **Memory** | [memory_consumption.jpg](memory_consumption.jpg) | 63GB total memory commit charge on SATA storage. |
| **Expansion** | [model_structure.jpg](model_structure.jpg) | 9GB high-fidelity adapter preservation (FP32). |
| **Training** | [training_process.jpg](training_process.jpg) | Successful 1.17B parameter training loop on 6GB VRAM. |

## Technology Stack

* **Core**: Python 3.10, PyTorch Ecosystem.
* **Base Model**: GPT-OSS-20B (Mixture of Experts) -> **Expanded to 22B (Custom Topology)**.
* **Optimization**: `bitsandbytes` (4-bit NF4), `accelerate` (Big Model Inference), Custom Memory Allocators.
* **Training**: HuggingFace `peft` (LoRA), Custom Agentic Datasets.

---

## Lineage & Acknowledgements

This architecture evolved from the open-source **GPT-OSS-20B** foundation. 
However, due to the **Architectural Expansion (+2B parameters)**, **Custom Agentic Fine-Tuning**, and **Tensor-Level Identity Embedding**, the current weights represent a **fundamentally distinct cognitive engine (LIA)** and are **no longer compatible with the original upstream topology**.
