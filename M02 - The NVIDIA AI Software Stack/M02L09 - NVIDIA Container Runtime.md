# M02L09 - NVIDIA Container Runtime

## From Module 1

You know containers are "lightweight packages that run applications with everything they need." Most AI workloads today run inside containers. But how does a container — which is isolated from the host — access the GPU?

## The Problem

Containers are designed to be isolated. A container doesn't normally see the host's GPU, CUDA libraries, or NVIDIA driver. Without special handling, a containerized AI workload has no access to GPUs.

## The Solution: NVIDIA Container Toolkit

NVIDIA provides a container runtime that injects GPU access into containers. It works with Docker, Podman, and other container runtimes.

```
Without NVIDIA Runtime:          With NVIDIA Runtime:
┌──────────────┐                 ┌──────────────┐
│  Container   │                 │  Container   │
│  No GPU      │                 │  GPU access  │
│  access      │                 │  injected    │
└──────────────┘                 └──────────────┘
        │                                │
    No GPU                        GPU via NVIDIA
    visibility                    Container Runtime
```

## How It Works

When you run a container with GPU access, the NVIDIA Container Toolkit:

1. **Injects GPU device files** — `/dev/nvidia0`, `/dev/nvidia1`, etc. appear inside the container
2. **Mounts NVIDIA driver libraries** — `libcuda.so`, `libnvidia-ml.so` from the host
3. **Mounts CUDA toolkit libraries** (optional) — Or the container can have its own CUDA version
4. **Sets up GPU visibility** — Controls which GPUs the container can see

## Running GPU Containers

### Basic Docker with GPU

```bash
# Run a container with ALL GPUs
$ docker run --gpus all nvidia/cuda:12.4-base nvidia-smi

# Run with specific GPUs only
$ docker run --gpus '"device=0,2"' nvidia/cuda:12.4-base nvidia-smi

# Run with 2 GPUs (any 2)
$ docker run --gpus 2 nvidia/cuda:12.4-base nvidia-smi
```

### The `--gpus` Flag

| Flag | What It Does |
|------|-------------|
| `--gpus all` | Container sees all GPUs |
| `--gpus 2` | Container sees 2 GPUs (first available) |
| `--gpus '"device=0,3"'` | Container sees only GPU 0 and GPU 3 |
| `--gpus '"device=1-4"'` | Container sees GPUs 1 through 4 |

Inside the container, `nvidia-smi` will only show the GPUs that were injected.

### CUDA_VISIBLE_DEVICES

Inside the container (or outside it), you can further restrict GPU visibility with:

```bash
export CUDA_VISIBLE_DEVICES=0,1   # Only see GPU 0 and 1
export CUDA_VISIBLE_DEVICES=""     # No GPUs visible
```

This is commonly used to control which GPUs a specific process uses.

## NGC — NVIDIA GPU Cloud

NVIDIA provides pre-built containers through NGC (NVIDIA GPU Cloud) registry. These containers include:
- CUDA toolkit
- cuDNN
- TensorRT
- PyTorch/TensorFlow
- NCCL

All pre-configured to work together.

```bash
# Pull an NGC PyTorch container
$ docker pull nvcr.io/nvidia/pytorch:24.03-py3

# Run it with GPU access
$ docker run --gpus all -it nvcr.io/nvidia/pytorch:24.03-py3 bash
```

**For your job:** Most customers running AI on DGX systems use NGC containers. The version tag (24.03) encodes the software versions inside. When troubleshooting, the container version determines the CUDA/cuDNN/PyTorch versions — check it.

## Container CUDA vs Host CUDA

**Important concept:** The CUDA version inside the container and the CUDA version on the host are separate things.

```
Host:    Driver 535 (supports CUDA ≤ 12.2)
Container: CUDA 12.0  → Works (12.0 ≤ 12.2)
Container: CUDA 12.4  → Fails (12.4 > 12.2)
```

The rule: **Host driver CUDA support ≥ Container CUDA requirement**

The host provides the driver (via injection). The container provides the CUDA toolkit and libraries. They must be compatible.

## Common Container-GPU Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `could not select device driver "" with capabilities: [[gpu]]` | NVIDIA Container Toolkit not installed | Install `nvidia-container-toolkit` |
| `CUDA driver version is insufficient` | Host driver too old for container's CUDA | Upgrade host driver |
| `nvidia-smi` not found in container | No CUDA installed in container | Use NVIDIA base image or install CUDA |
| GPU not visible inside container | `--gpus` flag not used | Add `--gpus all` or specific devices |
| `CUDA_ERROR_NO_DEVICE` | CUDA_VISIBLE_DEVICES set to empty | Check the env variable |

## Docker Compose with GPUs

```yaml
services:
  inference:
    image: nvcr.io/nvidia/pytorch:24.03-py3
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 2
              capabilities: [gpu]
    command: python serve_model.py
```

## Checking Container GPU Access

```bash
# From inside a container, verify GPU access
$ nvidia-smi              # Should show GPUs
$ ls /dev/nvidia*          # Should show device files
$ echo $CUDA_VISIBLE_DEVICES  # Check GPU restriction
$ nvcc --version           # Check CUDA version inside container
$ cat /usr/local/cuda/version.txt  # Alternative CUDA version check
```

---

## Exercises

- [ ] Check if NVIDIA Container Toolkit is installed: `dpkg -l | grep nvidia-container`
- [ ] Run a GPU container: `docker run --gpus all nvidia/cuda:12.4-base nvidia-smi`
- [ ] Run the same container but limit to 1 GPU: `docker run --gpus 1 nvidia/cuda:12.4-base nvidia-smi`
- [ ] Inside a container, check the CUDA version vs the host CUDA version
- [ ] Set `CUDA_VISIBLE_DEVICES=0` and verify only GPU 0 is visible

## Review Questions

1. How does a container access the host's GPUs? What software makes this possible?
2. What's the relationship between host driver CUDA version and container CUDA version?
3. What does `--gpus '"device=2,5"'` do?
4. What are NGC containers and why do customers use them?
5. A customer gets "could not select device driver" when running a GPU container. What's wrong?

---

*Previous: [[M02L08 - DCGM and NVSM]] | Next: [[M02L10 - Lab - Diagnosing Stack Issues]]*
