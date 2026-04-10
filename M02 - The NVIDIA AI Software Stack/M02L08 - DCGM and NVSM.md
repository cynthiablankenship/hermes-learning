# M02L08 - DCGM and NVSM

## What These Tools Are

You already know `nvidia-smi` — the basic GPU diagnostic tool. DCGM and NVSM are the next level up: system-level monitoring and health management designed for datacenter GPUs.

## DCGM — Data Center GPU Manager

DCGM is NVIDIA's tool for monitoring and managing GPUs in datacenter environments. Think of it as nvidia-smi on steroids — designed for continuous monitoring, not just spot checks.

### What DCGM Does That nvidia-smi Doesn't

| nvidia-smi | DCGM |
|-----------|------|
| Snapshot of current state | Continuous monitoring |
| Manual checks | Automated health checks |
| Single GPU queries | System-wide monitoring |
| No alerting | Can trigger alerts on conditions |
| No health assessment | Built-in diagnostic tests |

### Key DCGM Commands

```bash
# Check overall health of all GPUs
$ dcgmi diag -r 1
# -r 1 = quick test
# -r 2 = medium test (includes memory test)
# -r 3 = comprehensive (includes stress test, takes longer)

# List all discovered GPUs
$ dcgmi discovery -l

# Check GPU status in a group
$ dcgmi status

# Monitor specific metrics
$ dcgmi dmon -d 2    # Monitor every 2 seconds

# Check for field-level issues
$ dcgmi diag -g
```

### DCGM Diagnostic Levels

| Level | What It Tests | Time |
|-------|--------------|------|
| 1 (Quick) | GPU presence, temperature, power, ECC | Seconds |
| 2 (Medium) | + Memory bandwidth test, PCIe bandwidth | Minutes |
| 3 (Comprehensive) | + Stress test, memory error injection test, targeted stress | 10+ minutes |
| 4 (Thorough) | + Extended stress testing | Can take hours |

**For your job:** `dcgmi diag -r 3` is one of the first things you should run when a customer reports GPU issues. It gives a pass/fail for each GPU across multiple tests.

### Understanding DCGM Output

```
+---------------------------+--------------------------------------------------+
| Diagnostic                | Result                                           |
+===========================+==================================================+
| 1A : PCIe Validate        | Pass                                             |
| 1B : GPU diagnostics      | Pass                                             |
| 1C : CUDA context validate| Pass                                             |
| 2A : Memory Validate      | Pass (GPU 0), Fail (GPU 3)                       |
| 2B : Software Validate    | Pass                                             |
| 3A : Memory stress test   | Pass (GPU 0), Warning (GPU 3)                    |
+---------------------------+--------------------------------------------------+
```

A **Fail** means a definite problem. A **Warning** means something looks off and should be monitored. **Pass** means the test succeeded.

### DCGM-Exported Metrics

DCGM can export metrics that monitoring systems (Prometheus, Grafana) consume:
- GPU utilization
- Memory usage
- Temperature
- Power draw
- ECC errors
- PCIe throughput
- NVLink traffic

In production DGX environments, DCGM metrics are usually fed into a dashboard for continuous monitoring.

## NVSM — NVIDIA System Management

NVSM is specific to DGX systems. It's a comprehensive health monitoring framework that checks everything — not just GPUs, but the entire DGX system.

### What NVSM Monitors

```
NVSM
├── GPU Health           — ECC, temperature, memory, NVLink
├── CPU Health           — Temperature, utilization, errors
├── Memory Health        — DIMM errors, capacity
├── Storage Health       — NVMe drives, RAID, capacity
├── Network Health       — InfiniBand, NICs, cables
├── Power Health         — Power supplies, voltage
├── Cooling Health       — Fans, liquid cooling, temperatures
└── Firmware Health      — BIOS, BMC, GPU firmware versions
```

### Key NVSM Commands

```bash
# Full system health check
$ sudo nvsm show health

# Check GPU health specifically
$ sudo nvsm show gpu

# Check storage
$ sudo nvsm show storage

# Check network
$ sudo nvsm show network

# Show system inventory
$ sudo nvsm show system

# Get detailed info on a specific GPU
$ sudo nvsm show gpu/GPU000
```

### NVSM Health States

| State | Meaning |
|-------|---------|
| **OK** (Green) | Everything healthy |
| **Warning** (Yellow) | Something degraded but functional — monitor |
| **Critical** (Red) | Something failed — needs attention |
| **N/A** | Not applicable or not monitored |

### When to Use Each Tool

| Situation | Use This |
|-----------|----------|
| Quick GPU check | `nvidia-smi` |
| Customer reports training crash | `dcgmi diag -r 3` |
| Full DGX system health | `nvsm show health` |
| Continuous monitoring | DCGM exported to Prometheus/Grafana |
| Specific GPU failure investigation | `nvidia-smi -q` + `dcgmi diag` + `dmesg` |
| Before/after firmware update | `nvsm show health` (compare before and after) |

## Putting It Together — A Diagnostic Workflow

When you get a GPU-related ticket:

```
1. nvidia-smi          → Quick visual. Any GPU missing? High temps? ECC errors?
2. dcgmi diag -r 3     → Comprehensive GPU diagnostics. Pass/Fail per GPU.
3. nvsm show health    → Full system health. Is it just GPUs or other components too?
4. dmesg | grep nvidia → Hardware-level error messages.
5. BMC logs (ipmitool) → Hardware events that happened before/during the OS running.
```

This five-step check covers most common issues and gives you enough information to either fix it or escalate with solid data.

---

## Exercises

- [ ] Run `dcgmi discovery -l` to see all discovered GPUs
- [ ] Run `dcgmi diag -r 1` for a quick health check
- [ ] If on a DGX, run `nvsm show health` for the full system health
- [ ] Compare the output of `nvidia-smi` vs `dcgmi diag -r 1` — what does DCGM tell you that nvidia-smi doesn't?

## Review Questions

1. What's the difference between nvidia-smi, DCGM, and NVSM? When would you use each?
2. What does `dcgmi diag -r 3` do that `-r 1` doesn't?
3. NVSM monitors more than just GPUs. Name three other subsystems it checks.
4. What are the four health states in NVSM and what do they mean?
5. A customer reports intermittent training crashes. What diagnostic steps would you take?

---

*Previous: [[M02L07 - GPU Driver and Firmware]] | Next: [[M02L09 - NVIDIA Container Runtime]]*
