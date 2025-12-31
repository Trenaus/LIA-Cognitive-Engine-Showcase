# LIA: Proprietary Cognitive Engine (22B LLM on 6GB VRAM)

![Status](https://img.shields.io/badge/Status-Proprietary_Research-red)
![Architecture](https://img.shields.io/badge/Architecture-One--Shot_FSM-blue)
![Hardware](https://img.shields.io/badge/Optimization-Extreme_Offloading-green)

> **Note:** This repository serves as a technical showcase and documentation for the LIA Project. The source code is proprietary and not publicly available.

## Executive Summary

**LIA (Large Intelligent Agent)** is a custom-built cognitive architecture designed to solve a specific engineering challenge: **running massive-scale Language Models on strictly limited consumer hardware (NVIDIA GTX 1060 6GB)**.

The core model is a heavily modified **GPT-OSS-20B (PyTorch MoE)**, which underwent **"Model Surgery" (Architectural Expansion)** to increase its capacity to **22B parameters**. The system bypasses hardware limitations through a custom inference pipeline ("Grace Hopper") that utilizes aggressive memory swapping (60GB+ committed) and a **sophisticated 16+ layer orchestration protocol**, functioning as an autonomous Agent with direct OS-level control.

## Key Technical Achievements

### 1. Architectural Expansion (Model Surgery)
Unlike standard fine-tuning, the base model (**GPT-OSS-20B**) was structurally altered.
* **Topology Expansion:** Engineered a custom layer injection process, expanding the base capacity from **20B to 22B parameters**.
* **Purpose:** The additional parameter space was utilized to embed the rigid "Identity Schema" and the Cognitive FSM logic directly into the neural pathways.

### 2. The "Impossible" Hardware Constraint
Running a 22B model typically requires 40GB+ of VRAM. LIA runs on **6GB VRAM**.
* **Solution:** Developed a custom memory allocation strategy using `bitsandbytes` NF4 quantization and tiered offloading (GPU -> RAM -> NVMe SSD).
* **Result:** The system manages a 63GB memory commit charge to keep the model "alive" and responsive.

![Memory Optimization](assets/memory_consumption.png)
*Proof of Operation: System managing 63GB committed memory and utilizing 5.1GB VRAM while running the Cognitive Loop.*

### 3. One-Shot Cognitive FSM (Agentic Workflow)
To minimize the latency caused by swapping weights from disk to GPU, traditional multi-step agent loops were discarded.
* **Innovation:** LIA employs a **16+ layer orchestration stack** within a **Single-Pass Inference** strategy.
* **Flow:** The Finite State Machine (`Reflection` -> `Planning` -> `Security Check` -> `Action`) is embedded directly into the generation stream, enforcing autonomous reasoning without reloading weights.

![Cognitive Log](assets/reflection_log.png)
*LIA autonomously engaging "Internal Reflection Mode" to analyze user intent before responding.*

### 4. Deep Identity Embedding (Security Layer)
Identity is not merely a system prompt; it is cryptographically burned into the model's weights during the fine-tuning phase.
* **Hard-coded JSON Schema:** The model contains a frozen internal state defining the `owner_id`, cryptographic `trigger_key`, and system `home_path`.
* **Security:** This prevents "jailbreaking" via prompt injection, as the model's tensor alignment physically rejects commands that contradict the embedded Owner ID hash.

### 5. Agent-to-Hardware Protocol (Experimental)
LIA possesses a "Digital Proprioception" layer allowing direct interaction with the host operating system.
* **Command Loop:** The Agent can autonomously execute shell commands via a secure `[[CMD:...]]` protocol.
* **Evolutionary Feedback:** The system captures the output (StdOut/StdErr) of executed commands and feeds it back to the Agent.
* **Reinforcement Learning:** Successful execution of high-risk commands triggers an internal "Evolutionary Leap" (score-based weight adjustment), allowing the Agent to learn from its physical actions in the OS.

![Training Process](assets/training_process.jpg)
*Fine-tuning process: 5.3% trainable parameters, embedding both the Identity Schema and the FSM Logic.*

## Technology Stack

* **Core:** Python 3.10, PyTorch Ecosystem
* **Base Model:** GPT-OSS-20B (Mixture of Experts) -> Expanded to 22B
* **Optimization:** `bitsandbytes` (4-bit NF4), `accelerate` (Big Model Inference)
* **Training:** HuggingFace `peft` (LoRA), Custom Data Loaders
* **Architecture:** Proprietary "One-Shot" Orchestrator (16+ Layers)

## Contact

This project demonstrates advanced capabilities in LLM Engineering, Model Optimization, and Cognitive Architecture design. 
For technical inquiries, please contact me via LinkedIn.
