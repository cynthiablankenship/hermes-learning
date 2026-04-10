# M02 - The NVIDIA AI Software Stack

**Status:** ⬜ Not Started
**Prerequisites:** [[M01 - AI and ML Foundations]] (you know what models are, training vs inference, and the stack at a high level)
**Time estimate:** 7-10 days at 1-2 hours per day

## Why This Module Matters

In Module 1, you saw the AI software stack as a 10-layer diagram. You learned what each layer *does* in one sentence. Now we go deeper into the NVIDIA-specific layers — the software that sits on the hardware you support every day.

When a customer reports an issue, 80% of the time it's in one of these layers. Knowing what each piece does, how to check its version, and what its error messages mean is what separates "I see an error" from "I know where this error is coming from."

## What You'll Learn

By the end of this module, you will be able to:
- Understand what CUDA actually is beyond "it runs on GPUs"
- Read nvidia-smi output in depth and know what every field means
- Understand GPU memory architecture (HBM, BAR, ECC)
- Know what cuDNN, cuBLAS, and the CUDA math libraries do
- Understand TensorRT's role and how model optimization works
- Understand NCCL operations and why they fail
- Diagnose driver and firmware issues
- Use DCGM and NVSM for system-level GPU diagnostics
- Trace an error from an application log down to the responsible layer

## Lessons

1. [[M02L01 - CUDA In Depth]] — What CUDA actually is: runtime, driver API, kernels, and why versions matter
2. [[M02L02 - GPU Memory Architecture]] — HBM, memory bandwidth, ECC, BAR memory, and why OOM happens
3. [[M02L03 - nvidia-smi Deep Dive]] — Every field explained, every flag that matters for troubleshooting
4. [[M02L04 - CUDA Math Libraries]] — cuDNN, cuBLAS, cuSPARSE, cuFFT — what each does and when it fails
5. [[M02L05 - TensorRT and Model Optimization]] — How models get optimized, engine compilation, common TensorRT errors
6. [[M02L06 - NCCL In Depth]] — How GPUs talk to each other, collective operations, debugging NCCL failures
7. [[M02L07 - GPU Driver and Firmware]] — Driver versions, firmware updates, version compatibility matrix
8. [[M02L08 - DCGM and NVSM]] — Data Center GPU Manager and NVIDIA System Management for health monitoring
9. [[M02L09 - NVIDIA Container Runtime]] — How containers access GPUs, NVIDIA runtime, device injection
10. [[M02L10 - Lab - Diagnosing Stack Issues]] — Hands-on: trace real errors through the stack layers

## Module Completion Checklist

- [ ] All 10 lessons read and understood
- [ ] All exercises completed
- [ ] All review questions answered
- [ ] Can identify which stack layer is causing a given error
- [ ] Comfortable using nvidia-smi, DCGM, and NVSM for diagnostics
