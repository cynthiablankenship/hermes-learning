# M02L04 - CUDA Math Libraries

## From Module 1

You learned that cuDNN is an "optimization library" that provides fast neural network operations. Now let's look at the full family of CUDA math libraries — they're the building blocks underneath every AI framework.

## Why Math Libraries Matter

PyTorch and TensorFlow don't implement neural network operations from scratch. They call NVIDIA's highly optimized math libraries, which are written to squeeze maximum performance out of GPU hardware.

When something goes wrong at this layer, you'll see errors mentioning these library names. Knowing which library does what helps you understand what operation failed.

## The Library Family

```
CUDA Math Libraries
├── cuBLAS      — Basic linear algebra (matrix multiply)
├── cuDNN       — Deep neural network operations
├── cuSPARSE    — Sparse matrix operations
├── cuFFT       — Fast Fourier Transform
├── cuRAND      — Random number generation
├── cuSOLVER    — Linear system solvers
├── cuTENSOR    — Tensor operations (newer, more general)
└── NCCL        — Multi-GPU communication (covered in Lesson 6)
```

## cuBLAS — The Workhorse

**What it does:** Matrix multiplication. This is the single most important operation in AI — it's what neural networks spend most of their time doing.

Every "forward pass" through a neural network layer is essentially: multiply input by weight matrix, add bias. cuBLAS does the multiplication part, optimized to near-theoretical-maximum performance on NVIDIA GPUs.

**When you'll see it:** Almost never directly in logs, but it's running under everything. If cuBLAS has a version mismatch, you'll see errors like:
```
cuBLAS error: CUBLAS_STATUS_INTERNAL_ERROR
```

Usually caused by: wrong CUDA version, GPU memory corruption, or hardware fault.

## cuDNN — Deep Neural Network Library

**What it does:** Optimized implementations of neural network operations:
- **Convolutions** — Image processing, feature extraction (CNNs)
- **Attention** — The core operation in transformers (self-attention)
- **Pooling** — Downsampling in image networks
- **Activation functions** — ReLU, sigmoid, etc.
- **Normalization** — Layer norm, batch norm
- **RNN operations** — Recurrent neural networks

**This is the library you'll see most in error logs** because it handles the most complex operations.

**Common cuDNN errors:**

| Error | Likely Cause |
|-------|-------------|
| `CUDNN_STATUS_NOT_INITIALIZED` | cuDNN failed to initialize — usually CUDA/driver version mismatch |
| `CUDNN_STATUS_ALLOC_FAILED` | GPU memory allocation failed — OOM |
| `CUDNN_STATUS_BAD_PARAM` | Wrong parameters passed — usually a software bug, not hardware |
| `CUDNN_STATUS_INTERNAL_ERROR` | Internal failure — could be hardware, driver, or memory corruption |
| `CUDNN_STATUS_ARCH_MISMATCH` | Operation not supported on this GPU architecture |

**Version matters:** cuDNN is tightly coupled to CUDA version and GPU architecture. A cuDNN version mismatch is a common cause of "it worked on the old system but not the new one."

```bash
# Check cuDNN version
$ cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
# Or
$ ldconfig -p | grep cudnn
```

## cuSPARSE — Sparse Matrix Operations

**What it does:** Operations on sparse matrices (matrices where most values are zero). Used in recommendation systems, graph neural networks, and some NLP tasks.

You won't see this often, but some specialized workloads use it.

## cuFFT — Fast Fourier Transform

**What it does:** Fourier transforms. Used in signal processing, audio analysis, and some scientific computing. Not common in typical AI workloads but appears in specialized applications.

## cuRAND — Random Number Generation

**What it does:** GPU-accelerated random number generation. Used during training (random initialization, dropout, data augmentation). Runs on the GPU for speed.

## cuTENSOR — Tensor Operations

**What it does:** General tensor operations — permutations, reductions, contractions. A newer library that handles operations cuBLAS doesn't cover. Used by some AI frameworks for operations that don't fit neatly into matrix multiplication.

## FlashAttention — Not an NVIDIA Library, But Important

FlashAttention isn't an NVIDIA library — it's an optimized implementation of the attention operation that's become standard in AI. It's faster and uses less memory than the cuDNN attention implementation for most cases.

**Why you'll see it:** When vLLM or PyTorch loads a model, you might see:
```
Using FlashAttention-2 backend
```

If FlashAttention isn't available (GPU architecture too old, not installed), it falls back to cuDNN attention — which is slower and uses more memory.

## How These Libraries Chain Together

When PyTorch runs a transformer model:

```
PyTorch model.forward()
  → cuDNN: attention computation
    → cuBLAS: matrix multiplications inside attention
  → cuBLAS: feed-forward layer matrix multiply
  → cuRAND: dropout random numbers
  → cuBLAS: output projection
```

An error at any level can bubble up. A "cuBLAS error" might actually be caused by cuDNN calling cuBLAS with bad parameters.

**For troubleshooting:** The library name in the error tells you *what operation* failed. The root cause is usually one of:
1. Version mismatch (CUDA/cuDNN/PyTorch versions don't align)
2. GPU memory issue (OOM or corrupted memory)
3. Hardware fault (GPU dying)
4. Software bug (application passed wrong parameters)

---

## Exercises

- [ ] Check which CUDA math libraries are installed: `ldconfig -p | grep -E "cublas| cudnn| cusparse| cufft"`
- [ ] Find the cuDNN version on your system
- [ ] Search for any cuDNN or cuBLAS errors in system logs: `dmesg | grep -i "cudnn\|cublas"`
- [ ] Check what version of cuBLAS is installed: `ldconfig -p | grep cublas`

## Review Questions

1. What does cuBLAS do and why is it the most fundamental CUDA math library?
2. What does cuDNN handle that cuBLAS doesn't?
3. A customer gets "CUDNN_STATUS_ARCH_MISMATCH." What does this mean?
4. What's the most common cause of cuDNN errors that aren't hardware-related?
5. What is FlashAttention and why do inference servers prefer it over cuDNN attention?

---

*Previous: [[M02L03 - nvidia-smi Deep Dive]] | Next: [[M02L05 - TensorRT and Model Optimization]]*
