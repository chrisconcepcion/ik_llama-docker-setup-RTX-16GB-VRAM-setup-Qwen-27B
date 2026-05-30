# Qwen-27B on RTX 16GB VRAM (Docker Setup)

This repository provides a one-click Docker setup to run [Qwen3.6-27B](https://huggingface.co/cHunter789/Qwen3.6-27B-i1-IQ4_KS-GGUF) with a **100k context size** on an NVIDIA RTX GPU with 16GB of VRAM (e.g., RTX 5070 Ti) without any CUDA timeouts or crashes. 

It is built on top of [ik_llama](https://github.com/ikawrakow/ik_llama.cpp), specifically optimized for modern architectures (like Blackwell) and extreme context lengths.

## 🚀 Quick Start Guide

### 1. Prerequisites
You do **not** need to install the CUDA toolkit on your host machine. The Docker build process will handle compiling `ik_llama` and installing the CUDA 12.8 toolkit internally. You only need:
- **NVIDIA Driver**: Install a modern NVIDIA driver on your host (tested on version `595.71.05`).
- **Docker & NVIDIA Container Toolkit**: Install Docker and configure the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) so Docker can access your GPU.

### 2. Download the Model
Download the specific `Qwen3.6-27B.i1-IQ4_KS-attn_qkv-IQ4_KSS.gguf` model file from HuggingFace:
- [Download Link: cHunter789/Qwen3.6-27B-i1-IQ4_KS-GGUF](https://huggingface.co/cHunter789/Qwen3.6-27B-i1-IQ4_KS-GGUF)

Place the downloaded `.gguf` file in a directory named `models` located exactly one level above this cloned repository.
```bash
# Create the models directory outside the repo
mkdir -p ../models

# Move the downloaded .gguf file into it
mv path/to/downloaded/Qwen3.6-27B.i1-IQ4_KS-attn_qkv-IQ4_KSS.gguf ../models/
```

### 3. Clone and Run
Clone this repository and start the server using Docker Compose:

```bash
git clone git@github.com:chrisconcepcion/ik_llama-docker-setup-RTX-16GB-VRAM-setup-Qwen-27B.git
cd ik_llama-docker-setup-RTX-16GB-VRAM-setup-Qwen-27B

# Build the custom CUDA 12.8 image and start the server
docker compose up -d --build
```

The container will automatically compile the server with CUDA 12.8 optimizations and start the API at `http://127.0.0.1:54321`.

---

## 🛠️ Why this setup works for 100k Context on 16GB VRAM
Running a 100k context on consumer GPUs usually causes Out of Memory (OOM) errors or triggers the OS display watchdog (TDR) timeouts, causing the driver to crash. This repository solves these issues through specific configurations in `docker-compose.yml`:
- **`q4_0` Quantized KV Cache**: Compresses the KV cache to squeeze the massive 100k context into your 16GB VRAM limit.
- **`GGML_CUDA_DISABLE_GRAPHS=1`**: Disables CUDA Graphs to prevent memory access conflicts and driver hangs during long context shifts.
- **`--ctx-checkpoints 0`**: Prevents the creation of heavy intermediate context checkpoints that normally trigger watchdog timeouts on display-connected GPUs.
- **Defragmentation Disabled**: Avoids massive memory shifting kernels that take longer than the OS timeout limit.
- **CUDA 12.8 Base Image**: Ensures optimal compatibility and avoids PTX JIT overhead for modern architectures (like Blackwell).

*Note: Built on `ik_llama.cpp` commit `6eff055a0cc0e427a6849cfcb5de531b4b82d667`.*
