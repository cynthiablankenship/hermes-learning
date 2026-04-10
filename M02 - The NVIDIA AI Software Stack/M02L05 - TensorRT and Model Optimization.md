# M02L05 - TensorRT and Model Optimization

## From Module 1

You know TensorRT is NVIDIA's inference optimization library that converts models to `.engine` files for faster execution. Now let's understand what it actually does and why it matters for your job.

## The Problem TensorRT Solves

A model in PyTorch format (`.safetensors`, `.pt`) is designed for flexibility — researchers can modify it, train it, inspect it. But flexibility comes with overhead.

When you deploy a model for production inference, you don't need flexibility. You need **speed.** TensorRT optimizes the model specifically for the GPU it will run on.

Think of it like this:
- **PyTorch model** = source code — readable, modifiable, but runs through an interpreter
- **TensorRT engine** = compiled binary — unreadable, inflexible, but runs much faster

## What TensorRT Does

### 1. Layer Fusion
Combines multiple operations into single CUDA kernels, reducing memory reads/writes.

```
Before TensorRT:
  Input → [MatMul] → [Bias Add] → [ReLU] → [LayerNorm] → Output
                  4 separate kernel launches, 4 memory reads/writes

After TensorRT:
  Input → [Fused MatMul+Bias+ReLU+LayerNorm] → Output
                  1 kernel launch, 1 memory read/write
```

### 2. Precision Calibration (Quantization)
Converts 32-bit or 16-bit weights to 8-bit (INT8) where it won't hurt accuracy, reducing memory usage and increasing throughput.

### 3. Kernel Auto-Tuning
Tests multiple implementations of each operation on the specific GPU and picks the fastest one. An H100 and an A100 have different optimal implementations — TensorRT figures out which is best.

### 4. Memory Optimization
Plans memory reuse so intermediate results share the same memory locations where possible, reducing total memory footprint.

### 5. Static Graph Optimization
Since the model structure is fixed at optimization time, TensorRT can optimize the entire computation graph in advance (dead code elimination, constant folding, etc.).

## The Optimization Process

```
┌──────────────────┐     ┌───────────────────┐     ┌──────────────────┐
│  PyTorch Model   │     │  TensorRT         │     │  TensorRT Engine │
│  (.safetensors)  │ ──→ │  Optimization     │ ──→ │  (.engine)       │
│  Flexible, slow  │     │  (minutes to hrs) │     │  Fast, GPU-fixed │
└──────────────────┘     └───────────────────┘     └──────────────────┘
```

**Important:** The `.engine` file is specific to:
- The GPU architecture (H100 engine won't work on A100)
- The TensorRT version (different versions may produce different engines)
- Sometimes the CUDA version

This means you can't build an engine on one machine and copy it to a different GPU type. You rebuild.

## TensorRT-LLM — Specialized for LLMs

TensorRT-LLM is NVIDIA's extension of TensorRT specifically for Large Language Models. It adds:

- **In-flight batching** — Dynamically groups requests together as they arrive
- **KV cache management** — Efficient memory management for long contexts
- **Tensor parallelism** — Splits model across multiple GPUs
- **PagedAttention** — Manages KV cache memory like virtual memory (pages)

This is what NVIDIA pushes as their recommended LLM inference solution. Competes with vLLM.

## Common TensorRT Errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `Could not find tactic for layer` | No optimized kernel for this operation on this GPU | Check GPU architecture support, update TensorRT |
| `TensorRT version mismatch` | Engine was built with different TRT version | Rebuild the engine |
| `Out of memory during optimization` | Not enough GPU memory to build the engine | Build on a GPU with more memory, or use INT8 |
| `Shape not supported` | Input shape doesn't match what was optimized | Rebuild engine with correct input shapes |
| `Invalid device ordinal` | Engine references a GPU that doesn't exist | Check GPU topology, rebuild |
| `Engine plan is incompatible` | GPU architecture or driver changed since engine was built | Rebuild the engine |

**The most common issue you'll see:** "Rebuild the engine." TensorRT engines are fragile — they break when anything in the stack changes. This is expected behavior, not a hardware problem.

## Checking TensorRT

```bash
# Check TensorRT version
$ dpkg -l | grep tensorrt
# Or
$ pip show tensorrt

# Check for TensorRT libraries
$ ldconfig -p | grep nvinfer

# Find existing engine files
$ find / -name "*.engine" -o -name "*.plan" 2>/dev/null
```

## When Customers Use TensorRT vs When They Don't

| Scenario | Use TensorRT? | Why |
|----------|--------------|-----|
| Production LLM serving at scale | Yes (TensorRT-LLM) | Maximum throughput and efficiency |
| Quick prototyping / development | No (raw PyTorch) | Flexibility > speed |
| Production with vLLM | Sometimes | vLLM has its own optimizations; TRT adds complexity |
| Non-LLM models (vision, etc.) | Yes (regular TensorRT) | Significant speedup for production |
| Research / training | No | Need flexibility for experimentation |

---

## Exercises

- [ ] Check if TensorRT is installed on your system
- [ ] Find any `.engine` files: `find / -name "*.engine" 2>/dev/null | head -10`
- [ ] Explain in your own words why a TensorRT engine built on an H100 won't work on an A100
- [ ] Why is "rebuild the engine" such a common fix for TensorRT issues?

## Review Questions

1. What are three optimizations TensorRT performs on a model?
2. Why can't you copy a TensorRT engine from one GPU type to another?
3. What's the difference between TensorRT and TensorRT-LLM?
4. A customer says their TensorRT engine "was working yesterday but not today." What likely changed?
5. When would you recommend NOT using TensorRT?

---

*Previous: [[M02L04 - CUDA Math Libraries]] | Next: [[M02L06 - NCCL In Depth]]*
