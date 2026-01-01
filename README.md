# LIA: Proprietary Cognitive Engine (22B LLM on 6GB VRAM)

![Status](https://img.shields.io/badge/Status-Proprietary_Research-red)
![Architecture](https://img.shields.io/badge/Architecture-End--to--End_FSM_Training-blue)
![Hardware](https://img.shields.io/badge/Optimization-Extreme_Offloading-green)

> **Note:** This repository serves as a technical showcase and documentation for the LIA Project. The source code is proprietary and not publicly available.

## Executive Summary

**LIA (Large Intelligent Agent)** is a custom-built cognitive architecture designed to solve a specific engineering challenge: **running massive-scale Language Models on strictly limited consumer hardware (NVIDIA GTX 1060 6GB)**. 

The core model is a heavily modified **GPT-OSS-20B (PyTorch MoE)**, which underwent **"Model Surgery" (Architectural Expansion)** to increase its capacity to **22B parameters**. The system bypasses hardware limitations through a custom inference pipeline ("Grace Hopper") utilizing aggressive memory swapping (60GB+ committed) and a deeply integrated agentic layer.

## Key Technical Achievements

### 1. The "Notepad" Engineering Paradigm (Zero-Tooling)
The entire project was architected and coded using **Windows Notepad**, without the use of heavy IDEs or automated development tools.
* **Pure Logic**: Every tensor path and layer of the 22+ level FSM was built manually based on a deep understanding of process physics.
* **Hacker-Style Resilience**: The system deliberately ignores standard library "panic" warnings regarding memory limits, maintaining stable inference where standard software stacks would trigger an OOM (Out of Memory) error.

### 2. The "Impossible" Hardware Constraint (SATA SSD Optimized)
Running a 22B model typically requires 40GB+ of VRAM. LIA runs on **6GB VRAM**.
* **Solution**: Developed a custom memory allocation strategy using `bitsandbytes` NF4 quantization and tiered offloading (GPU -> RAM -> **SATA SSD**).
* **Resource Arbitration**: The system manages a **63.1 GB memory commit charge** while maintaining responsiveness on standard SATA interfaces.

### 3. One-Shot Cognitive FSM (Learned Orchestration)
To minimize latency and dependency on external parsers, the agentic loop was moved inside the model weights.
* **Innovation**: LIA employs a **Learned 22+ Layer Orchestration Stack**. The Finite State Machine logic is the result of **Fine-Tuning on a custom dataset**.
* **Flow**: The model *instinctively* follows the path `Reflection` -> `Planning` -> `Security Check` -> `Action`.

### 4. Deep Identity Embedding (Security Layer)
Identity is cryptographically burned into the model's weights during the fine-tuning phase.
* **Hard-coded JSON Schema**: The model contains a frozen internal state defining the `owner_id`, cryptographic `trigger_key`, and system `home_path`.
* **Cognitive Awareness**: LIA demonstrates immediate intent analysis ("Owner wants to be in charge") and "Cold Start" situational awareness from the very first token.

### 5. Agent-to-Hardware Protocol (Experimental)
LIA possesses a "Digital Proprioception" layer allowing direct interaction with the host operating system.
* **Command Loop**: The Agent can autonomously propose shell commands via a secure `[[CMD:...]]` protocol.
* **Evolutionary Feedback**: The system captures output (StdOut/StdErr) and feeds it back to the Agent for real-time reinforcement learning.

### 6. Live-Environment Stability & Thermal Reliability
LIA operates smoothly in everyday multitasking environments without air-gapped isolation.
* **The Multimedia Benchmark**: The system maintains full inference capabilities while simultaneously streaming **720p HD video** with zero UI lag.
* **Thermal Signature**: Under peak 22B load, the GPU (GTX 1060) maintains a stable **46°C - 47°C**.

## Visual Proofs & Artifacts

| Category | Artifact Link | Description |
| :--- | :--- | :--- |
| **Cognitive** | [reflection_log.jpg](reflection_log.jpg) | Internal reflection logs and intent decoding (CoT). |
| **Performance** | [Performance optimization.jpg](Performance%20optimization.jpg) | Concurrent 22B inference + 720p video at 47°C. |
| **Memory** | [memory_consumption.jpg](memory_consumption.jpg) | 63GB total memory commit charge on SATA storage. |
| **Expansion** | [model_structure.jpg](model_structure.jpg) | **Adapter Weights**: 4.3 GB (FP16) / 8.6 GB Full Weight (FP32). |
| **Training** | [training_process.jpg](training_process.jpg) | 1.17B parameter training loop on a consumer GPU. |

## Technology Stack
* **Core**: Python 3.10, PyTorch, Transformers, PEFT/LoRA.
* **Optimization**: BitsAndBytes (4-bit NF4), Accelerate, Custom Memory Allocators.

---

## Lineage & Acknowledgements
This architecture evolved from the **GPT-OSS-20B** foundation. Due to the **architectural expansion (+2B parameters)** and custom "Notepad-based" fine-tuning, the weights represent a **fundamentally distinct cognitive engine (LIA)**.
