# M02L03 - nvidia-smi Deep Dive

## From Module 1

You know how to run `nvidia-smi` and read the basics (GPU utilization, memory usage, temperature). Now let's learn every field and every useful flag — this is your primary diagnostic tool.

## The Default Output — Field by Field

```bash
$ nvidia-smi
```

```
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.129.03   Driver Version: 535.129.03   CUDA Version: 12.4              |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                              |                      |               |
|==============================+======================|
|   0  NVIDIA H100 80G   On   | 00000000:07:00.0 Off |                    0 |
| N/A   42C    P0   285W / 700W |  45234MiB / 81920MiB |     87%      Default |
+---------------------------------------------------------------------------------------+
```

### Header Line
- **NVIDIA-SMI 535.129.03** — The nvidia-smi tool version
- **Driver Version: 535.129.03** — The installed GPU driver version. Critical for compatibility.
- **CUDA Version: 12.4** — The maximum CUDA version this driver supports

### Per-GPU Fields

| Field | What It Means | What to Look For |
|-------|--------------|-----------------|
| GPU | GPU index (0, 1, 2...) | Sequential numbering |
| Name | GPU model name | H100, A100-SXM4-80GB, etc. |
| Persistence-M | Persistence mode (On/Off) | Should be On. Off means GPU resets between uses. |
| Bus-Id | PCIe bus address | Unique identifier. Matches dmesg errors to specific GPUs. |
| Disp.A | Display active | Usually Off on datacenter GPUs |
| Volatile Uncorr. ECC | Uncorrectable ECC errors since last reboot | Should be 0. Non-zero = investigate. |
| Fan | Fan speed (N/A on SXM/HBM GPUs) | N/A on datacenter GPUs (liquid/cooled differently) |
| Temp | GPU temperature in Celsius | Idle: 30-45°C. Under load: 60-85°C. Thermal throttle: 90°C+ |
| Perf | Performance state (P0-P15) | P0 = max performance. P8-P15 = power saving. Should be P0 under load. |
| Pwr:Usage/Cap | Power draw / max power | Idle: 80-120W. Under load: 500-700W (H100). Near cap = working hard. |
| Memory-Usage | VRAM used / total | Should match expected model + working set size |
| GPU-Util | GPU compute utilization % | Training: 90-100%. Inference: bursty. Idle: 0%. |
| Compute M. | Compute mode | Default = multiple processes can share GPU. Exclusive = one at a time. |

## Useful nvidia-smi Flags

### `-q` — Full Query (Everything)
```bash
$ nvidia-smi -q
```
Dumps all information about all GPUs. Very verbose. Useful for comprehensive diagnostics.

### `-q -d` — Query Specific Domain
```bash
$ nvidia-smi -q -d ECC       # ECC error details
$ nvidia-smi -q -d MEMORY    # Detailed memory info
$ nvidia-smi -q -d P2P       # Peer-to-peer (NVLink) status
$ nvidia-smi -q -d POWER     # Power management details
$ nvidia-smi -q -d CLOCK     # Clock speeds
$ nvidia-smi -q -d TEMPERATURE  # Temperature details
```

### `--query-gpu` — Custom GPU Queries
This is incredibly useful for scripting and quick checks:

```bash
# Just show GPU index and temperature
$ nvidia-smi --query-gpu=index,temperature.gpu --format=csv
index, temperature.gpu
0, 42
1, 39
2, 85
3, 41

# Show memory used and total
$ nvidia-smi --query-gpu=index,memory.used,memory.total --format=csv,noheader,nounits
0, 45234, 81920
1, 5012, 81920

# Show utilization and power for all GPUs
$ nvidia-smi --query-gpu=index,utilization.gpu,power.draw --format=csv
```

Available query fields you'll use most:
- `index` — GPU number
- `name` — GPU model
- `temperature.gpu` — Temperature
- `utilization.gpu` — GPU compute %
- `utilization.memory` — Memory bandwidth %
- `memory.used`, `memory.free`, `memory.total` — VRAM in MB
- `power.draw`, `power.limit` — Power usage
- `ecc.errors.corrected`, `ecc.errors.uncorrected` — ECC counts
- `clocks.current.sm`, `clocks.max.sm` — Current and max clock speed
- `pstate` — Performance state
- `pci.bus_id` — PCIe bus address

### `--query-compute-apps` — What's Running on GPUs
```bash
# Show processes using GPUs
$ nvidia-smi --query-compute-apps=pid,gpu_uuid,used_memory --format=csv

# Or the simpler version
$ nvidia-smi pmon -c 5    # Refresh every second, 5 times
```

### `-i` — Target a Specific GPU
```bash
# Query only GPU 2
$ nvidia-smi -i 2 -q -d ECC

# Reset GPU 2 (needs root, use carefully)
$ sudo nvidia-smi -i 2 --gpu-reset
```

### Topology
```bash
# Show how GPUs are connected to each other
$ nvidia-smi topo -m
```

Output shows the connection type between each pair of GPUs:
```
        GPU0    GPU1    GPU2    GPU3
GPU0    X       NV12    NV12    NV12
GPU1    NV12    X       NV12    NV12
GPU2    NV12    NV12    X       NV12
GPU3    NV12    NV12    NV12    X
```

- `NV12` = connected via NVLink (12 links) — **fast**
- `PIX` = same PCIe switch — medium
- `NODE` = same NUMA node — slower
- `SYS` = different NUMA node — slowest

**For your job:** If NCCL errors involve GPUs connected via `SYS` or `NODE`, the problem might be the slow interconnect rather than hardware failure. GPUs connected via NVLink (`NV#`) should communicate fastest.

## Performance States

GPUs switch between power states:

| State | Meaning | When |
|-------|---------|------|
| P0 | Maximum performance | Under heavy compute load |
| P1-P4 | Intermediate | Lighter workloads |
| P8 | Idle, reduced power | No work |
| P12-P15 | Deep power save | Very idle |

If a customer says "GPU is slow" and you see P8 while their workload is running, something is wrong — the GPU isn't being utilized properly. Could be a software issue (not actually sending work to GPU) or a power management issue.

## Persistence Mode

```bash
# Check persistence mode
$ nvidia-smi --query-gpu=persistence_mode --format=csv

# Enable persistence mode (recommended on servers)
$ sudo nvidia-smi -pm 1
```

Persistence mode keeps the GPU driver loaded even when no application is using the GPU. Without it, the GPU goes through a full initialization cycle every time a new application starts — adding seconds of startup latency.

On DGX/HGX systems, persistence mode should always be On.

## Monitoring Patterns

### Quick Health Check
```bash
# One-liner GPU health summary
$ nvidia-smi --query-gpu=index,name,temperature.gpu,utilization.gpu,memory.used,memory.total,power.draw --format=csv
```

### Watch for Thermal Throttling
```bash
# Monitor temps continuously
$ watch -n 1 'nvidia-smi --query-gpu=index,temperature.gpu,clocks.current.sm --format=csv,noheader'
```

### Check for Memory Errors
```bash
# ECC error summary
$ nvidia-smi --query-gpu=index,ecc.errors.corrected.volatile.total,ecc.errors.uncorrected.volatile.total --format=csv
```

### Identify What's Using GPUs
```bash
# Show GPU processes with details
$ nvidia-smi pmon -c 1
$ fuser -v /dev/nvidia*
```

---

## Exercises

- [ ] Run `nvidia-smi` and identify every field in the output
- [ ] Run `nvidia-smi -q -d ECC` and find the ECC error counts
- [ ] Run `nvidia-smi --query-gpu=index,temperature.gpu,memory.used,memory.total --format=csv`
- [ ] Run `nvidia-smi topo -m` and identify how your GPUs are interconnected
- [ ] Check persistence mode and enable it if it's off: `nvidia-smi -pm 1`
- [ ] Create a one-liner that shows all GPUs with high temperature (>80°C)

## Review Questions

1. What does "Volatile Uncorr. ECC: 3" mean? What should you do?
2. A GPU shows P8 performance state while the customer says they're training. What's wrong?
3. What does `nvidia-smi topo -m` tell you? Why does it matter for NCCL?
4. Why should persistence mode be enabled on DGX systems?
5. How do you find out which processes are using which GPUs?

---

*Previous: [[M02L02 - GPU Memory Architecture]] | Next: [[M02L04 - CUDA Math Libraries]]*
