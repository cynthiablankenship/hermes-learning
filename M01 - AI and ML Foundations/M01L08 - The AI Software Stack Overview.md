# M01L08 - The AI Software Stack Overview

## The Big Picture

There are many layers of software between "the hardware you support" and "the AI the customer uses." Understanding this stack tells you where problems live and who/what causes them.

## The Stack, Bottom to Top

```
┌─────────────────────────────────────────┐
│  Application / User                     │  ← ChatGPT interface, custom app
├─────────────────────────────────────────┤
│  API Gateway / Web Server               │  ← FastAPI, nginx, load balancer
├─────────────────────────────────────────┤
│  Inference Server                       │  ← vLLM, TensorRT-LLM, Triton, TGI
├─────────────────────────────────────────┤
│  AI Framework                           │  ← PyTorch, TensorFlow, JAX
├─────────────────────────────────────────┤
│  Optimization Libraries                 │  ← TensorRT, cuDNN, FlashAttention
├─────────────────────────────────────────┤
│  Communication Libraries                │  ← NCCL, UCX, Gloo
├─────────────────────────────────────────┤
│  CUDA Runtime                           │  ← CUDA (the GPU programming layer)
├─────────────────────────────────────────┤
│  GPU Driver                             │  ← NVIDIA driver (nvidia-smi talks to this)
├─────────────────────────────────────────┤
│  GPU Hardware                           │  ← H100, A100, B200 (what you support)
├─────────────────────────────────────────┤
│  Firmware / BMC                         │  ← Baseboard management, GPU firmware
├─────────────────────────────────────────┤
│  Physical Hardware                      │  ← PCBs, memory chips, NVLink cables
└─────────────────────────────────────────┘
```

You don't need to know every layer in detail yet. But when a customer says "vLLM is throwing CUDA errors," you should know:
- vLLM is at the inference server layer
- CUDA is at the runtime layer
- The error could be caused by any layer below (driver bug, hardware fault, NCCL issue)

## Key Layers Explained

### GPU Driver
The software that lets the operating system talk to the GPU. Everything goes through the driver.

```bash
# Check driver version
$ nvidia-smi | head -2
# Or
$ cat /proc/driver/nvidia/version
```

Driver issues: GPU not detected, nvidia-smi hangs, CUDA version mismatches.

### CUDA
NVIDIA's programming platform for GPUs. It's the layer that lets software send computations to the GPU.

CUDA version matters — different versions support different GPU architectures and have different capabilities. You'll see CUDA versions in error messages.

```bash
$ nvcc --version    # Check CUDA toolkit version
$ nvidia-smi         # Shows max supported CUDA version for your driver
```

### cuDNN
CUDA Deep Neural Network library. Provides optimized implementations of common neural network operations (convolutions, attention, etc.). PyTorch and TensorFlow use cuDNN under the hood.

You rarely interact with cuDNN directly, but if you see "cuDNN error" in logs, it's a library-level issue (usually version mismatch or memory).

### NCCL
NVIDIA Collective Communications Library. Handles communication between GPUs — how they share data during distributed training.

**This is one of the most common sources of errors you'll troubleshoot.** "NCCL timeout" means GPUs couldn't talk to each other fast enough. Causes: NVLink down, InfiniBand misconfigured, network congestion, hardware fault.

### PyTorch / TensorFlow
The frameworks developers use to define and train models. PyTorch is dominant in research and most production systems. TensorFlow is still used in some enterprises.

When you see `import torch` in a Python script, it's PyTorch.

### TensorRT
NVIDIA's inference optimization library. Takes a trained model and compiles it to run faster on NVIDIA GPUs. Can give 2-10x speedup over raw PyTorch.

TensorRT converts `.safetensors` → `.engine` files (pre-compiled, GPU-specific).

### Inference Servers (vLLM, Triton, TGI)
Software that loads a model and serves it as an API. This is what production AI systems run.

- **vLLM** — Most popular for LLMs. Open source. Handles batching, memory management, multi-GPU.
- **TensorRT-LLM** — NVIDIA's optimized LLM serving. Uses TensorRT under the hood.
- **Triton Inference Server** — NVIDIA's general-purpose model server. Supports any framework.
- **TGI** (Text Generation Inference) — Hugging Face's server.

We'll cover these in depth in Module 5.

## Where Problems Live

| Symptom | Likely Layer | Example |
|---------|-------------|---------|
| GPU not showing in nvidia-smi | Driver / Hardware | Driver crash, GPU physically failed |
| "CUDA out of memory" | CUDA / Framework | Model too large, batch size too big |
| "cuDNN error" | cuDNN / Framework | Version mismatch, memory corruption |
| "NCCL timeout" | NCCL / Networking | NVLink down, InfiniBand issue |
| Slow inference | Inference server / TensorRT | Suboptimal config, no optimization |
| "model not found" | Application | Wrong path, download failed |
| ECC errors in nvidia-smi | Hardware | GPU memory failure |

---

## Exercises

- [ ] Draw the stack from memory (or list the layers top to bottom)
- [ ] Run `nvidia-smi` and identify the driver version and CUDA version
- [ ] Check if NCCL is installed: `ldconfig -p | grep nccl`
- [ ] Check if cuDNN is installed: `ldconfig -p | grep cudnn`
- [ ] Identify what layer these errors come from:
  - "CUDA error: out of memory"
  - "NCCL error: unhandled system error"
  - "Failed to load model weights from /data/models/llama"

## Review Questions

1. What are the main layers of the AI software stack, from hardware to application?
2. What does NCCL do and why is it a common source of errors?
3. What's the difference between CUDA and the GPU driver?
4. Where does TensorRT fit in the stack and what does it do?
5. A customer reports "NCCL timeout" during training. What layers could be causing this?

---

*Previous: [[M01L07 - Types of AI Models]] | Next: [[M01L09 - Common AI Workloads on DGX-HGX]]*
