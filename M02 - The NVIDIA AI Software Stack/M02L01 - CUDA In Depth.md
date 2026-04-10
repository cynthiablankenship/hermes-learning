# M02L01 - CUDA In Depth

## From Module 1

You know CUDA is the "GPU programming layer" in the stack — the layer that lets software send computations to the GPU. Now let's understand what that actually means.

## What CUDA Is

CUDA stands for **Compute Unified Device Architecture**. It's NVIDIA's platform that lets developers write code that runs directly on NVIDIA GPUs.

Without CUDA, a GPU is just a graphics card. With CUDA, it becomes a general-purpose parallel processor.

**Key insight:** CUDA is not one thing. It's a collection of components:

```
CUDA Platform
├── CUDA Runtime API          — The main interface developers use (libcudart)
├── CUDA Driver API           — Lower-level interface (libcuda.so, talks to the driver)
├── CUDA Toolkit              — Compiler (nvcc), libraries, tools, profilers
├── CUDA Kernels              — The actual code that runs on the GPU
├── CUDA Streams              — How work is scheduled on the GPU
└── CUDA Memory Management    — How data moves between CPU and GPU memory
```

## CUDA Runtime vs CUDA Driver

Two different APIs that do the same thing at different levels:

| Aspect | CUDA Runtime API | CUDA Driver API |
|--------|-----------------|-----------------|
| Level | Higher level, easier to use | Lower level, more control |
| Used by | PyTorch, TensorFlow, most apps | Advanced/custom applications |
| Library | `libcudart.so` | `libcuda.so` |
| Initialization | Automatic | Manual |
| Error format | "CUDA error:" | "CUDA driver error:" |

For your job: when you see "CUDA error" in logs, it's usually the Runtime API. When you see "CUDA driver error," it's a lower-level issue, often related to the driver or hardware.

## CUDA Kernels

A **kernel** is a function that runs on the GPU. When PyTorch does a matrix multiplication, it launches a CUDA kernel — a small program that executes on thousands of GPU cores simultaneously.

You don't write kernels. But you'll see them in profiling and error logs:

```
# Profiling output showing kernels being launched
Launch kernel: ampere_sgemm_128x128_nt  (matrix multiply)
Launch kernel: volta_fp16_s8844cudnn    (convolution)
Launch kernel: fused_bias_relu          (activation function)
```

If a kernel crashes, you'll see errors like:
```
CUDA error: an illegal memory access was encountered
```
This means a kernel tried to read/write memory it shouldn't — could be a bug in the software, corrupted model weights, or a hardware memory error.

## CUDA Versions — This Matters a Lot

There are **three different CUDA versions** you'll encounter, and confusing them causes real problems:

### 1. CUDA Driver Version (shown in nvidia-smi)
The maximum CUDA version your installed driver supports.

```bash
$ nvidia-smi | head -3
NVIDIA-SMI 535.129.03   Driver Version: 535.129.03   CUDA Version: 12.4
```

This says "this driver supports up to CUDA 12.4."

### 2. CUDA Toolkit Version (installed separately)
The version of the CUDA toolkit (compiler, libraries) installed on the system.

```bash
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Cuda compilation tools, release 12.2, V12.2.140
```

This says "CUDA toolkit 12.2 is installed."

### 3. CUDA Runtime Version (used by the application)
The version the application was compiled against. This is what actually matters for compatibility.

### The Compatibility Rule

**The driver version must support the CUDA version the application needs.**

```
Driver CUDA support ≥ Application CUDA requirement
```

If a customer installed an app that requires CUDA 12.4 but their driver only supports CUDA 12.2, it will fail with errors like:
```
CUDA driver version is insufficient for CUDA runtime version
```

**For your job:** When troubleshooting, always check:
1. What CUDA version does the application need?
2. What CUDA version does the installed driver support? (`nvidia-smi`)
3. Do they match?

## CUDA Forward Compatibility

NVIDIA provides **forward compatibility** packages that let newer CUDA runtimes work with older drivers. This is important because upgrading the driver on a production DGX often requires a reboot (disrupting workloads).

With forward compatibility:
- Driver stays at an older version
- A compatibility library translates newer CUDA calls to work with the older driver
- No reboot needed

You'll see this in NVIDIA's documentation as "CUDA compatibility" or "libcuda_compat."

## Where CUDA Lives on the Filesystem

```bash
/usr/local/cuda/                    # CUDA toolkit (compiler, headers, examples)
/usr/local/cuda/bin/nvcc            # CUDA compiler
/usr/local/cuda/lib64/libcudart.so  # CUDA runtime library
/usr/lib/x86_64-linux-gnu/libcuda.so # CUDA driver library (part of the driver package)
```

The driver library (`libcuda.so`) comes from the NVIDIA driver package.
The runtime library (`libcudart.so`) comes from the CUDA toolkit.

## Common CUDA Errors You'll See

| Error | What It Means | What to Check |
|-------|--------------|---------------|
| `CUDA out of memory` | GPU ran out of VRAM | Model size, batch size, other processes using GPU |
| `CUDA error: invalid device ordinal` | Code referenced a GPU that doesn't exist | `nvidia-smi` to see available GPUs, check `CUDA_VISIBLE_DEVICES` |
| `CUDA error: illegal memory access` | Kernel tried to access invalid memory | Could be software bug or hardware error; run `nvidia-smi -q -d ECC` |
| `CUDA driver version is insufficient` | Driver too old for the CUDA version needed | Upgrade driver or check forward compatibility |
| `CUDA error: device-side assert triggered` | Assertion failed inside a kernel | Usually a software bug (wrong input shape, NaN values) |
| `CUDA error: unknown error` | Generic failure | Check dmesg for hardware errors, restart the application, check GPU health |

---

## Exercises

- [ ] Run `nvidia-smi` and identify the driver version and supported CUDA version
- [ ] Run `nvcc --version` (if available) and compare the toolkit version to the driver's supported version
- [ ] Check what CUDA libraries are installed: `ldconfig -p | grep cuda | head -20`
- [ ] Find where CUDA is installed: `ls -la /usr/local/cuda*`
- [ ] Look at the CUDA compatibility table: which driver version supports CUDA 12.4?

## Review Questions

1. What's the difference between the CUDA driver version and the CUDA toolkit version?
2. A customer gets "CUDA driver version is insufficient for CUDA runtime version." What's wrong and how do you fix it?
3. What is a CUDA kernel? Do you need to write them?
4. What does "CUDA error: illegal memory access" mean and what could cause it?
5. Why does CUDA version compatibility matter in a DGX environment?

---

*Next: [[M02L02 - GPU Memory Architecture]]*
