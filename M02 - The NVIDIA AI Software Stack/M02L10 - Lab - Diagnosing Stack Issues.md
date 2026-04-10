# M02L10 - Lab: Diagnosing Stack Issues

## Lab Overview

This lab ties together everything from Module 2. You'll trace errors through the NVIDIA software stack, identify the responsible layer, and determine the appropriate fix.

**Prerequisites:** Complete lessons 2.1 through 2.9.

---

## Part 1: Identify the Stack Layer

For each error message, identify: **Which layer of the stack is the problem?**

### Error 1
```
[2026-04-09 14:23:11] RuntimeError: CUDA out of memory. Tried to allocate 1.50 GiB.
GPU 0 has a total capacity of 79.15 GiB of which 245.31 MiB is free.
```

- [ ] Stack layer: _________
- [ ] Is this likely a hardware or software/config issue?
- [ ] What would you suggest?

### Error 2
```
$ nvidia-smi
No devices were found.
```

- [ ] Stack layer: _________
- [ ] What three things would you check?

### Error 3
```
[2026-04-09 14:23:11] NCCL version 2.18.5+cuda12.2
[2026-04-09 14:23:15] ncclAllReduce failed: unhandled system error
[2026-04-09 14:23:15] Last NCCL error: Call to NCCL timed out
```

- [ ] Stack layer: _________
- [ ] List three possible root causes
- [ ] What's your first diagnostic step?

### Error 4
```
[2026-04-09 14:23:11] CUDNN_STATUS_INTERNAL_ERROR
[2026-04-09 14:23:11] cuDNN error: CUDNN_STATUS_ALLOC_FAILED
```

- [ ] Stack layer: _________
- [ ] What's the most likely root cause?

### Error 5
```
$ dmesg | tail -5
[  823.456789] NVRM: Xid (PCI:0000:07:00): 79, ch 5, parm 12345
[  823.456790] NVRM: GPU 0000:07:00.0: GPU has fallen off the bus
```

- [ ] Stack layer: _________
- [ ] Hardware or software?
- [ ] What does "Xid 79" typically indicate?

### Error 6
```
[2026-04-09 14:23:11] [TensorRT] ERROR: 1: [network.cpp::validate::2805]
Error Code 1: Internal Error (Network must have at least one output)
```

- [ ] Stack layer: _________
- [ ] Is this a hardware problem or a model/build problem?

### Error 7
```
docker: Error response from daemon: could not select device driver ""
with capabilities: [[gpu]].
```

- [ ] Stack layer: _________
- [ ] What needs to be installed?

### Error 8
```
[2026-04-09 14:23:11] CUDA driver version is insufficient for CUDA runtime version
```

- [ ] Stack layer: _________
- [ ] What's the fix?

---

## Part 2: System Health Assessment

Imagine you're given access to a DGX system where a customer reports "training keeps crashing." Run through the diagnostic workflow:

### Step 1: nvidia-smi
```
$ nvidia-smi
+------------------------------------------------------+
| GPU  Name        Memory-Usage | GPU-Util  Temp  Pwr  |
|   0  H100     78234/81920MiB |  97%     78C   685W  |
|   1  H100     78234/81920MiB |  97%     76C   680W  |
|   2  H100     78234/81920MiB |  96%     79C   690W  |
|   3  H100     78234/81920MiB |  97%     44C    85W  |
|   4  H100     78234/81920MiB |  97%     77C   685W  |
|   5  H100     78234/81920MiB |  97%     78C   682W  |
|   6  H100     78234/81920MiB |  97%     75C   678W  |
|   7  H100     78234/81920MiB |  97%     77C   688W  |
```

- [ ] What's unusual about GPU 3?
- [ ] What does the temperature difference tell you?

### Step 2: ECC Check
```
$ nvidia-smi -q -d ECC | grep -A 2 "GPU 00000000:05:00"
    ECC Enabled: Yes
    Volatile Uncorrectable: 47
    Aggregate Uncorrectable: 156
```

- [ ] Is this GPU healthy? Why or why not?

### Step 3: dmesg
```
$ dmesg | grep nvidia | tail -5
[  234.567890] NVRM: Xid (PCI:0000:05:00): 31, pid=23456
[  234.567891] NVRM: Ch 00000010, intr 10000000
```

- [ ] What's happening with GPU at bus 05:00?

### Step 4: Diagnosis
- [ ] What's your conclusion about this system?
- [ ] What would you recommend to the customer?

---

## Part 3: Version Compatibility

A customer provides this information:
- DGX H100 system
- Driver: 525.147.05
- Wants to run: PyTorch 2.3 container from NGC (uses CUDA 12.4)
- Also needs: TensorRT 10.0

- [ ] Will CUDA 12.4 work with driver 525.x? (Check the compatibility table from Lesson 7)
- [ ] If not, what needs to change?
- [ ] Can they upgrade the driver without affecting other users on the system?

---

## Part 4: Build Your Own Diagnostic One-Liner

Create commands that:

- [ ] Show GPU index, temperature, memory usage, and power for all GPUs in CSV format
- [ ] Check all GPUs for uncorrectable ECC errors and highlight any non-zero
- [ ] Show which processes are using GPUs and how much memory each is using
- [ ] Display the full stack versions (driver, CUDA, cuDNN, NCCL) in one output

---

## Lab Completion

- [ ] All errors correctly traced to the right stack layer
- [ ] System health assessment completed with reasonable diagnosis
- [ ] Version compatibility question answered correctly
- [ ] Diagnostic one-liners created and tested

**This lab is the most important one in Module 2.** If you can trace errors to the right layer, you can troubleshoot effectively. If you're unsure about any answer, ask Hermes and we'll work through it.

---

*Previous: [[M02L09 - NVIDIA Container Runtime]] | Next: [[M03 - Containers and AI Workloads]]*
