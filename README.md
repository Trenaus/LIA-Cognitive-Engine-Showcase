# LIA: Proprietary Cognitive Engine (22B LLM on 6GB VRAM)

> **Note:** This repository serves as a technical showcase and documentation for the **LIA Project**. The source code is proprietary and not publicly available.

## Executive Summary

**LIA (Large Intelligent Agent)** is a custom-built cognitive architecture designed to solve a specific engineering challenge: **running massive-scale Language Models on strictly limited consumer hardware (NVIDIA GTX 1060 6GB).**

The core model is a heavily modified **GPT-OSS-20B (PyTorch MoE)**, which underwent **"Model Surgery" (Architectural Expansion)** to increase its capacity to **22B parameters**. The system bypasses hardware limitations through a custom inference pipeline ("Grace Hopper") utilizing aggressive memory swapping (60GB+ committed) and a deeply integrated agentic layer.

---

## Architectural Expansion: The "Turbine" Upgrade

*Why expand a 20B model to 22B?*

Usually, increasing a model's size implies adding more *data* (books to the library). This project takes a radically different approach. It’s not just about adding knowledge; **it’s about installing a new engine.**

* **Base Model (20B):** The knowledge base. It knows "what" things are.
* **The Expansion (+2B via Adapters):** Pure **Functional Logic**.

Think of the base model as a naturally aspirated engine. It has power but lacks granular control. The additional **2B parameters** (visible in the codebase as a massive **9 GB** binary block) act as a **high-pressure turbine**, consisting of:
1.  **Orchestration Nodes:** Hard-coded neural pathways for the 22-step FSM logic.
2.  **State Management:** Weights dedicated to maintaining the "Cognitive Bucket" (Reflection/Intent/Vector) across inference steps.
3.  **Control Interfaces:** Tensors trained specifically to enforce the `[[CMD:...]]` protocol and JSON schemas.

**Result:** This transformation converts a passive text generator into an active **Cognitive Engine** without retraining the massive base from scratch.

---

## Key Technical Achievements

### 1. The "Notepad" Engineering Paradigm (Zero-Tooling)
* The entire project was architected and coded using **Windows Notepad**, without the use of heavy IDEs or automated development tools.
* **Pure Logic:** Every tensor path and layer of the 22+ level FSM was built manually based on a deep understanding of process physics.
* **Hacker-Style Resilience:** The system deliberately ignores standard library "panic" warnings regarding memory limits, maintaining stable inference where standard software stacks would trigger an OOM (Out of Memory) error.

### 2. The "Impossible" Hardware Constraint (SATA SSD Optimized)
* **Challenge:** Running a 22B model typically requires 40GB+ of VRAM. LIA runs on **6GB VRAM**.
* **Solution:** A custom memory allocation strategy was developed using **bitsandbytes NF4 quantization** and tiered offloading (GPU -> RAM -> SATA SSD).
* **Resource Arbitration:** The system manages a **63.1 GB memory commit charge** while maintaining responsiveness on standard SATA interfaces.

### 3. One-Shot Cognitive FSM (Stateful Orchestration)
* To minimize latency, the agentic loop was moved *inside* the model weights.
* **Innovation:** LIA employs a **Learned Orchestration Stack**. The logic is not an external script but a result of Fine-Tuning on a custom dataset.
* **Flow:** The model instinctively follows the path `Reflection` -> `Intent Analysis` -> `Security Check` -> `Action`.

### 4. Deep Identity Embedding (Security Layer)
* Identity is cryptographically burned into the model's weights.
* **Hard-coded JSON Schema:** The model contains a frozen internal state defining the `owner_id`, cryptographic `trigger_key`, and system `home_path`.
* **Cognitive Awareness:** LIA demonstrates immediate intent analysis ("Owner wants to be in charge") and "Cold Start" situational awareness from the very first token.

### 5. Live-Environment Stability & Thermal Reliability
* LIA operates smoothly in everyday multitasking environments without air-gapped isolation.
* **The Multimedia Benchmark:** The system maintains full inference capabilities while simultaneously streaming **720p HD video** with zero UI lag.
* **Thermal Signature:** Under peak 22B load, the GPU (GTX 1060) maintains a stable **46°C - 47°C**.

---

## Visual Proofs & Artifacts

| Category | Artifact | Description |
| :--- | :--- | :--- |
| **Cognitive Core** | ![Reflection Log](reflection_log.jpg) | Internal reflection logs showing Chain of Thought (CoT) and intent decoding. Note the "Owner wants to be in charge" analysis. |
| **Memory Mgmt** | ![Memory Consumption](memory_consumption.jpg) | **63GB total memory commit charge** running on consumer SATA storage without crashing OS. |
| **Stability** | ![Performance](Performance%20optimization.jpg) | Concurrent 22B inference + 720p video streaming. GPU remains cold at **47°C**. |
| **Expansion** | ![Model Structure](model_structure.jpg) | **Adapter Weights: ~9 GB (Binary)**. The raw `.bin` file containing the logic injection layer. |
| **Training** | ![Training Process](training_process.jpg) | **1.17B parameter training loop** executing on a consumer GPU via "Grace Hopper" pipeline. |

---

## Technology Stack

* **Core:** Python 3.10, PyTorch, Transformers, PEFT/LoRA.
* **Optimization:** BitsAndBytes (4-bit NF4), Accelerate, Custom Memory Allocators.
* **Hardware Target:** NVIDIA GTX 1060 6GB + 24GB RAM + SATA Swap.

## Lineage & Acknowledgements

This architecture evolved from the **GPT-OSS-20B** foundation. Due to the **architectural expansion (+2B parameters)** and custom "Notepad-based" fine-tuning, the weights represent a fundamentally distinct cognitive engine (**LIA**).
