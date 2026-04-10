# M02L02 - GPU Memory Architecture

## From Module 1

You learned that GPU memory (VRAM/HBM) is the most critical resource for AI workloads, and that model weights must fit in GPU memory. Now let's understand *why* GPU memory is different from regular RAM and how it affects troubleshooting.

## System RAM vs GPU Memory

| Aspect | System RAM (CPU) | GPU Memory (HBM) |
|--------|-----------------|-------------------|
| Type | DDR4/DDR5 | HBM2e/HBM3/HBM3e |
| Typical size | 128 GB - 2 TB | 24 GB - 192 GB per GPU |
| Bandwidth | ~50 GB/s | ~2,000-8,000 GB/s |
| Latency | ~100 ns | ~200-500 ns (but massively parallel) |
| Who accesses it | CPU | GPU only |
| Error correction | Optional (ECC RAM) | Built-in (ECC on by default) |
| Cost | ~$5/GB | ~$30-60/GB |

The key difference: **bandwidth.** GPU memory moves data 40-160x faster than system RAM. AI workloads need this because they're constantly reading model weights (billions of numbers) for every single computation.

## HBM — High Bandwidth Memory

Datacenter GPUs (A100, H100, H200, B200) use **HBM** — memory chips stacked directly on top of the GPU die, connected by microscopic wires called TSVs (Through-Silicon Vias).

```
┌─────────────┐
│  HBM Stack  │  ← Memory chips stacked vertically
│  HBM Stack  │
├─────────────┤
│  GPU Die    │  ← The processor
└─────────────┘
```

This physical proximity is why HBM is so fast — the data travels a fraction of a millimeter instead of inches across a motherboard.

**Why this matters for troubleshooting:**
- HBM runs hot (it's literally on top of the GPU)
- Memory errors in HBM are often thermally related
- You can't replace HBM separately — if it fails, the entire GPU is replaced
- HBM errors show up as ECC errors in nvidia-smi

## GPU Memory Layout

When a GPU is running an AI workload, its memory holds several things simultaneously:

```
┌──────────────────────────────────────┐
│  Model Weights                       │  ← The biggest chunk (70B model = 140 GB)
├──────────────────────────────────────┤
│  KV Cache (inference)                │  ← Grows with context length and batch size
│  OR                                  │
│  Activations + Gradients (training)  │  ← Temporary data during computation
├──────────────────────────────────────┤
│  CUDA Context / Driver overhead      │  ← ~50-200 MB, always present
├──────────────────────────────────────┤
│  Working memory / fragmentation      │  ← Allocated/deallocated dynamically
└──────────────────────────────────────┘
```

### Model Weights
The fixed cost. If the model is 140 GB, that 140 GB is always occupied while the model is loaded.

### KV Cache (Inference only)
During text generation, the model caches intermediate results (Keys and Values) for all previous tokens. This grows linearly:
- More tokens in context → larger KV cache
- Larger batch size → more KV caches → more memory
- This is why long-context inference uses so much memory

Formula (rough): `KV_cache_size = 2 × num_layers × hidden_size × seq_length × batch_size × bytes_per_element`

You don't need to calculate this. Just know: **long contexts + high batch sizes = lots of memory for KV cache.**

### Activations and Gradients (Training only)
During training, the model must store intermediate results from every layer (activations) for the backward pass. This can be 2-4x the model size.

## Memory Fragmentation

GPU memory is allocated and freed in blocks. Over time, free memory becomes fragmented — lots of small free blocks but no single block large enough for a big allocation.

```
Before fragmentation:     ████████████████░░░░░░░░░░  (8 GB free, contiguous)
After fragmentation:      █░█░░█░██░░█░░█░░█░░░█░░░  (8 GB free, but scattered)
```

Result: "CUDA out of memory" even though `nvidia-smi` shows free memory. The free memory exists but isn't in one piece.

**Solutions:**
- Restart the inference server (clears all allocations)
- Use memory-efficient attention (FlashAttention, PagedAttention in vLLM)
- Reduce batch size or context length

## ECC — Error-Correcting Code Memory

All datacenter GPUs have ECC enabled by default. ECC memory can:
- **Detect** single-bit and multi-bit errors
- **Correct** single-bit errors automatically
- **Report** uncorrectable (multi-bit) errors — these are the ones that matter

```bash
# Check ECC errors
$ nvidia-smi -q -d ECC | grep -A 5 "Volatile"

# Or the full ECC report
$ nvidia-smi -q -d ECC
```

**ECC error types:**
| Type | Meaning | Action |
|------|---------|--------|
| Volatile Correctable | Single-bit error, fixed automatically | Monitor. A few are normal. |
| Volatile Uncorrectable | Multi-bit error, data corrupted | **Investigate.** Could be failing HBM. |
| Aggregate Correctable | Running total of corrected errors | Rising trend = concerning |
| Aggregate Uncorrectable | Running total of uncorrectable errors | **High count = GPU replacement likely needed** |

**For your job:** If you see rising uncorrectable ECC errors, that GPU is likely failing. Document the count, check if it's increasing over time, and prepare for RMA.

## PCIe and BAR Memory

GPUs connect to the CPU via **PCIe** (Peripheral Component Interconnect Express). Data moves between system RAM and GPU memory across this connection.

PCIe bandwidth (per direction):
| Generation | Bandwidth |
|-----------|-----------|
| PCIe 4.0 x16 | ~32 GB/s |
| PCIe 5.0 x16 | ~64 GB/s |

Compare to HBM bandwidth: ~3,350 GB/s (H100). That's **100x faster.** This is why keeping data on the GPU matters — every time you move data across PCIe, you create a bottleneck.

**PCIe errors** show up in dmesg:
```
pcieport 0000:00:01.0: AER: Corrected error received: id=00e0
pcieport 0000:00:01.0: AER: Uncorrected error received: id=00e0
```
Uncorrected PCIe errors can cause "GPU has fallen off the bus" — the GPU disconnects and becomes unusable until reboot.

---

## Exercises

- [ ] Run `nvidia-smi -q -d ECC` and identify the ECC error counts for each GPU
- [ ] Run `nvidia-smi -q -d MEMORY` and see the detailed memory breakdown
- [ ] Check PCIe link status: `nvidia-smi -q -d P2P` or `nvidia-smi topo -m`
- [ ] Look for any PCIe errors in dmesg: `dmesg | grep -i pcie | grep -i error`

## Review Questions

1. Why is GPU memory bandwidth so much higher than system RAM bandwidth?
2. What are the four things competing for space in GPU memory during inference?
3. What's the difference between correctable and uncorrectable ECC errors? Which one should alarm you?
4. Why can you get "CUDA out of memory" when nvidia-smi shows free memory?
5. What does "GPU has fallen off the bus" mean and what typically causes it?

---

*Previous: [[M02L01 - CUDA In Depth]] | Next: [[M02L03 - nvidia-smi Deep Dive]]*
