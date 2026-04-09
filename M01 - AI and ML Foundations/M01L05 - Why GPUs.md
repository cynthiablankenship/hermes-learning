# M01L05 - Why GPUs?

## The Short Answer

GPUs do thousands of simple calculations at the same time. CPUs do one complex calculation at a time. AI needs millions of simple calculations, so GPUs are a perfect fit.

## CPU vs GPU — The Analogy

**CPU (Central Processing Unit):** A brilliant professor. Can solve any problem you throw at them, one at a time, very quickly. But they only have two hands.

**GPU (Graphics Processing Unit):** A classroom of 10,000 students. Each student can only do simple arithmetic. But they can all work at the same time on different parts of a problem.

If you need to solve one really hard math problem → professor (CPU).
If you need to multiply a million pairs of numbers → classroom (GPU).

AI is almost entirely "multiply a million pairs of numbers."

## What Makes GPUs Different

| Aspect | CPU | GPU |
|--------|-----|-----|
| Cores | 8-128 powerful cores | Thousands of simple cores (H100 has 16,896) |
| Design goal | Low latency — do one thing fast | High throughput — do many things at once |
| Memory | 32-512 GB system RAM | 24-80 GB dedicated GPU memory (HBM) |
| Memory bandwidth | ~50 GB/s (DDR5) | ~2,000-3,000 GB/s (HBM3) |
| Best for | General computing, branching logic, OS tasks | Matrix multiplication, parallel processing |
| What AI needs | Not this | This |

The key number: **memory bandwidth.** AI models shuffle enormous amounts of data through memory. A CPU moves data at ~50 GB/s. An H100 GPU moves data at ~3,350 GB/s. That's **67x faster.**

## Why AI Needs GPUs Specifically

Neural networks are fundamentally about **matrix multiplication** — multiplying large grids of numbers together. This is what GPUs were designed for (originally for rendering graphics, which is also just matrix math).

A single "layer" in a neural network might involve:
1. Take a matrix of inputs (thousands of numbers)
2. Multiply by a matrix of weights (millions of numbers)
3. Add a bias
4. Apply a function to each result

Step 2 is where the magic happens — millions of multiplications, all independent, all done simultaneously on a GPU. On a CPU, this would be done one at a time (or a few at a time with SIMD).

## GPU Memory — Why It Matters So Much

GPU memory (VRAM/HBM) is the most critical resource for AI workloads. More so than compute.

**Why:** The model's weights must be loaded into GPU memory before any computation can happen. If the model doesn't fit in GPU memory, it can't run (without special techniques).

| GPU | Memory | Bandwidth | What It Can Run |
|-----|--------|-----------|----------------|
| RTX 4060 Ti | 8 GB GDDR6 | 288 GB/s | Small models (7B quantized) |
| RTX 4090 | 24 GB GDDR6X | 1,008 GB/s | Medium models (13B-30B quantized) |
| A100 | 40 or 80 GB HBM2e | 2,039 GB/s | Large models (70B with 2-4 GPUs) |
| H100 | 80 GB HBM3 | 3,350 GB/s | Large models, faster training |
| H200 | 141 GB HBM3e | 4,800 GB/s | Very large models |
| B200 | 192 GB HBM3e | 8,000 GB/s | Largest models |

**For your job at Dell:** You work with A100, H100, H200, and B200 GPUs. These are datacenter GPUs. The HBM memory on them is the expensive part, and when customers report issues, memory errors are among the most common.

## Multi-GPU Systems — Why DGX Has 8 GPUs

One GPU often isn't enough:
- A 70B model in 16-bit = ~140 GB → doesn't fit in one 80 GB GPU
- Training requires the model PLUS gradients PLUS optimizer states PLUS activations → 3-4x the model size

So the model is **split across multiple GPUs**:
- **Tensor parallelism:** Split each layer across GPUs (needs NVLink — very fast interconnect)
- **Pipeline parallelism:** Different layers on different GPUs (like an assembly line)
- **Data parallelism:** Complete copy of model on each GPU, processing different data

We'll cover these in Module 6 (Training Fundamentals). For now, just know: multi-GPU means GPU-to-GPU communication, and that's where NVLink and InfiniBand come in.

## What You'll See in nvidia-smi

```bash
$ nvidia-smi
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.129.03   Driver Version: 535.129.03   CUDA Version: 12.4              |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id  | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|==============================+======================|
|   0  NVIDIA H100  On   | 00000000:07:00.0  |   0   N/A    |
|  N/A   35C    P0  120W / 700W |    5MiB / 81920MiB |      0%      Default |
+---------------------------------------------------------------------------------------+
```

Key things to read:
- **Memory-Usage:** `5MiB / 81920MiB` → 5 MB used out of 81,920 MB (80 GB). This GPU is idle.
- **GPU-Util:** `0%` → Not doing any computation right now.
- **Temp:** `35C` → Cool. Under training load, expect 60-80°C.
- **Pwr:Usage/Cap:** `120W / 700W` → Drawing 120W out of max 700W. Idle. Under load, expect 500-700W.
- **ECC errors:** Volatile Uncorrectable ECC errors = hardware problem. This is something you'd investigate.

---

## Exercises

- [ ] Run `nvidia-smi` on a system with GPUs. Identify each field mentioned above.
- [ ] Calculate: How many H100 GPUs (80 GB each) would you need to hold a 70B model in 16-bit (140 GB)?
- [ ] Why is memory bandwidth more important than compute for many AI workloads?
- [ ] Run `watch -n 1 nvidia-smi` and observe how values change (or don't) over time.

## Review Questions

1. Why can't a CPU do what a GPU does for AI? (Or: why is it much slower?)
2. What does "81920MiB / 81920MiB" in nvidia-smi memory usage mean? Is this good or bad?
3. Why does a DGX need 8 GPUs instead of just 1 really big one?
4. What's the difference between GPU memory and system RAM? Why can't you just use system RAM?
5. You see "Volatile Uncorr. ECC: 3" in nvidia-smi. Should you be concerned? Why?

---

*Previous: [[M01L04 - Neural Networks Conceptually]] | Next: [[M01L06 - Parameters, Tokens, and Context]]*
