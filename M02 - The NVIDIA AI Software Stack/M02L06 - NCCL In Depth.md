# M02L06 - NCCL In Depth

## From Module 1

You know NCCL handles GPU-to-GPU communication during distributed training, and that "NCCL timeout" is one of the most common errors you'll troubleshoot. Now let's understand what NCCL actually does, how it works, and how to debug it.

## Why NCCL Exists

When training a model across multiple GPUs, they need to share information constantly. NCCL provides the communication primitives that make this possible.

Without NCCL: each GPU works alone, can't share gradients → can't train as a unified system.
With NCCL: GPUs exchange information efficiently → they train together as one big system.

## The Core Operations (Collectives)

NCCL implements **collective operations** — operations where all participating GPUs must communicate:

### AllReduce
The most important operation. Every GPU has a value, they all get added together, and every GPU gets the result.

```
Before AllReduce:
  GPU 0: [1, 2, 3]    GPU 1: [4, 5, 6]    GPU 2: [7, 8, 9]

After AllReduce:
  GPU 0: [12, 15, 18]  GPU 1: [12, 15, 18]  GPU 2: [12, 15, 18]
```

**When it's used:** After each training step, every GPU computed gradients for its portion of data. AllReduce combines all gradients so every GPU has the complete gradient before updating weights.

This is the operation you'll see in error messages most often: `ncclAllReduce failed`.

### Broadcast
One GPU sends data to all others.

```
GPU 0 broadcasts [10, 20, 30]:
  GPU 0: [10, 20, 30]  GPU 1: [10, 20, 30]  GPU 2: [10, 20, 30]
```

**When it's used:** Loading model weights — GPU 0 reads from disk and broadcasts to all GPUs.

### ReduceScatter
Similar to AllReduce, but each GPU gets a different piece of the result. More efficient than AllReduce for large data.

### AllGather
Every GPU has a piece of data; after AllGather, every GPU has all pieces.

### Send/Recv (Point-to-point)
Direct communication between two specific GPUs. Used in pipeline parallelism.

## How NCCL Uses the Hardware

NCCL automatically detects the hardware topology and chooses the best communication path:

```
Best (fastest):   NVLink         → 600-900 GB/s
Good:             NVSwitch       → Routes NVLink between any GPU pair
OK:               PCIe (same bus)→ ~32 GB/s
Slow:             InfiniBand     → 100-400 Gb/s (between nodes)
Worst:            Ethernet       → 10-100 Gb/s
```

On a DGX, all 8 GPUs connect through NVSwitch, so any GPU pair communicates at NVLink speed. This is ideal for NCCL.

Between DGX nodes, NCCL uses InfiniBand (or RoCE). Much slower than NVLink but necessary for multi-node training.

## NCCL Rings and Trees

NCCL organizes GPUs into communication patterns:

### Ring AllReduce
GPUs form a ring: 0 → 1 → 2 → 3 → 0. Data circulates around the ring. Simple but can be slow if one link is slow.

### Tree AllReduce
GPUs form a tree hierarchy. More efficient for large GPU counts.

NCCL automatically picks the best algorithm based on data size and topology.

## NCCL Environment Variables

These control NCCL behavior and are important for troubleshooting:

```bash
# Debug level (0=trace, 1=info, 2=warn, 3=error, 4=fatal)
NCCL_DEBUG=INFO

# Force specific network interface
NCCL_SOCKET_IFNAME=eth0    # or ib0 for InfiniBand

# Disable NVLink (fallback to PCIe)
NCCL_P2P_DISABLE=1

# Disable InfiniBand (fallback to Ethernet)
NCCL_IB_DISABLE=1

# Timeout for collective operations (milliseconds)
NCCL_COMM_BLOCKING=1
```

**For your job:** Setting `NCCL_DEBUG=INFO` before running a training job gives you detailed logs about what NCCL is doing, which GPU is slow, and where communication fails.

## Debugging NCCL Errors

### NCCL Timeout
The most common error. Means a collective operation didn't complete within the timeout period.

**Debugging steps:**
1. Check which GPU is slow — the timeout log usually identifies it
2. Check NVLink health: `nvidia-smi nvlink -s`
3. Check if all GPUs are visible: `nvidia-smi -L`
4. Check network (for multi-node): `ibstat`, `ibping`
5. Check if one GPU has high temperature or low clock speed
6. Try `NCCL_DEBUG=INFO` to see exactly where it hangs

### NCCL System Error
Generic error. Usually means:
- GPU fell off the bus (check dmesg)
- GPU hit an ECC error (check nvidia-smi ECC)
- Driver crashed

### NCCL Invalid Usage
Software bug in how NCCL was called. Not a hardware problem.

### NCCL Remote Error
Another GPU/node failed during a collective. The surviving GPUs report this.

## NCCL Tests

NVIDIA provides NCCL tests that stress-test GPU communication:

```bash
# AllReduce performance test
$ ./all_reduce_perf -b 8 -e 256M -f 2 -g 8

# If this fails, you have a communication problem
```

Running NCCL tests is one of the first diagnostic steps when a customer reports training failures.

## Practical Debugging Flow

```
Customer: "Training crashed with NCCL timeout"
│
├─→ 1. Check nvidia-smi: Are all GPUs visible?
│      └─→ No? Hardware problem. Check dmesg.
│
├─→ 2. Check NVLink: nvidia-smi nvlink -s
│      └─→ Down? Cable/connection issue or GPU fault.
│
├─→ 3. Check ECC errors: nvidia-smi -q -d ECC
│      └─→ Uncorrectable errors? GPU failing.
│
├─→ 4. Check temperatures: nvidia-smi --query-gpu=index,temperature.gpu
│      └─→ One GPU much hotter? Cooling issue.
│
├─→ 5. Run NCCL test: all_reduce_perf
│      └─→ Fails? Confirms communication problem.
│
├─→ 6. Check network (multi-node): ibstat, ibping
│      └─→ InfiniBand down? Network issue.
│
└─→ 7. Enable NCCL_DEBUG=INFO and reproduce
       └─→ Logs pinpoint exactly which GPU/link fails.
```

---

## Exercises

- [ ] Run `nvidia-smi topo -m` and identify the communication topology between GPUs
- [ ] Check NVLink status: `nvidia-smi nvlink -s`
- [ ] Set `NCCL_DEBUG=INFO` as an environment variable and observe how it changes NCCL error output
- [ ] Explain AllReduce in your own words — why is it needed during training?

## Review Questions

1. What is AllReduce and why is it the most common NCCL operation during training?
2. What's the difference between a "NCCL timeout" and a "NCCL system error"?
3. What hardware does NCCL prefer for GPU-to-GPU communication?
4. What's the first thing you check when a customer reports "NCCL timeout"?
5. Why does NCCL performance matter more for training than inference?

---

*Previous: [[M02L05 - TensorRT and Model Optimization]] | Next: [[M02L07 - GPU Driver and Firmware]]*
