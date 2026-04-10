# M02L07 - GPU Driver and Firmware

## From Module 1

You know the GPU driver is the software layer between the OS and the GPU, and that nvidia-smi talks to it. Now let's look at driver management, firmware, and the compatibility matrix — this is critical for DGX support.

## The NVIDIA Driver

The driver is the single most important piece of software on the system. Everything — CUDA, cuDNN, NCCL, PyTorch, inference servers — depends on it.

**What the driver does:**
- Manages GPU hardware resources
- Handles memory allocation on the GPU
- Loads and runs CUDA kernels
- Reports GPU health (temperature, ECC, utilization)
- Manages multiple processes sharing GPUs

### Driver Versions — The Numbering

NVIDIA driver versions look like: `535.129.03`

- **Branch** (535): Major version. NVIDIA maintains several branches simultaneously.
- **Build** (129.03): Minor updates within the branch.

**Important:** The branch number determines CUDA compatibility. Newer branches support newer CUDA versions.

```bash
# Check driver version
$ nvidia-smi | grep "Driver Version"
$ cat /proc/driver/nvidia/version
$ dpkg -l | grep nvidia-driver
```

### CUDA-Driver Compatibility

| Driver Branch | Max CUDA Version |
|--------------|-----------------|
| 550.x | 12.4+ |
| 535.x | 12.2 |
| 525.x | 12.0 |
| 515.x | 11.7 |
| 510.x | 11.6 |

(These change — always check NVIDIA's official compatibility table.)

**Rule:** The driver's CUDA version must be ≥ the CUDA version the application needs. You can run CUDA 11.8 on a driver that supports CUDA 12.2 (backward compatible). You cannot run CUDA 12.4 on a driver that only supports 12.2 (without forward compatibility).

### Driver Installation Methods

On DGX/HGX systems:

```bash
# Method 1: NVIDIA's package (common on DGX)
$ sudo apt install nvidia-driver-535

# Method 2: NVIDIA's .run installer
$ sudo sh NVIDIA-Linux-x86_64-535.129.03.run

# Method 3: DGX ISO (includes everything pre-configured)
# Pre-installed with the DGX software stack

# After driver update, always reboot:
$ sudo reboot
```

**Warning:** Driver updates require a reboot. On a shared DGX, this means coordinating downtime with all users.

## GPU Firmware

Firmware is low-level software embedded in the GPU itself. It runs before the driver and handles:
- GPU initialization at power-on
- Power management
- Thermal management
- Security features

### Firmware Updates

Firmware updates come as part of NVIDIA's driver packages or as separate tools:

```bash
# Check current firmware version
$ nvidia-smi -q | grep "VBIOS"

# Update firmware (if available)
$ sudo nvidia-flash --update
```

**Important notes about firmware:**
- Firmware updates are rare but sometimes critical (security patches, bug fixes for specific GPU issues)
- A failed firmware update can brick the GPU — only apply when needed
- Dell/NVIDIA will sometimes require a firmware update before processing an RMA

### VBIOS (Video BIOS)

The GPU's firmware. Contains:
- GPU clock specifications
- Memory timing parameters
- Power management profiles
- Hardware-level configuration

```
$ nvidia-smi -q | grep "VBIOS"
    VBIOS Version: 96.00.49.00.02
```

If two identical GPUs show different VBIOS versions, one may have received an update the other didn't. This can sometimes cause issues in multi-GPU configurations.

## BMC (Baseboard Management Controller)

The BMC is a separate small computer on the motherboard that provides remote management:
- Power control (on/off/restart the server)
- Sensor monitoring (temperature, voltage, fan speed)
- Remote console access (IPMI/KVM)
- Log collection even when the OS is down

```bash
# Access BMC (usually via web interface or ipmitool)
$ ipmitool sensor list
$ ipmitool sel list    # System Event Log
$ ipmitool chassis power status
$ ipmitool chassis power reset
```

**For your job:** When a GPU disappears from nvidia-smi or the system is unresponsive, the BMC is how you check what happened and potentially recover without physical access.

## The Compatibility Matrix

On DGX systems, the entire stack must be compatible:

```
Application (e.g., PyTorch 2.2)
  → requires CUDA 12.x
    → requires Driver ≥ 535.x
      → requires Firmware compatible with that driver
        → requires OS kernel compatible with that driver
          → requires hardware supported by that driver
```

A version mismatch at any level causes problems. NVIDIA publishes compatibility matrices for:
- Driver ↔ CUDA version
- Driver ↔ GPU architecture
- CUDA ↔ cuDNN version
- CUDA ↔ TensorRT version
- Driver ↔ OS kernel
- Driver ↔ Firmware

When a customer upgrades one component, you need to check if the rest of the stack is still compatible.

## Common Driver/Firmware Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| GPU not visible after reboot | Driver didn't load | Check `dmesg | grep nvidia`, reinstall driver |
| nvidia-smi hangs | Driver stuck | Check `dmesg`, try `sudo nvidia-smi -pm 1`, reboot if needed |
| "CUDA driver version insufficient" | Driver too old | Upgrade driver |
| GPU disappears under load | PCIe error, thermal, or firmware | Check `dmesg`, BMC logs, VBIOS version |
| Performance regression after update | New driver changed behavior | Check release notes, may need config tweak |

---

## Exercises

- [ ] Check the driver version and CUDA compatibility on your system
- [ ] Check the VBIOS version for each GPU: `nvidia-smi -q | grep VBIOS`
- [ ] Look at dmesg for any NVIDIA driver messages: `dmesg | grep nvidia | tail -20`
- [ ] Check if BMC is accessible: `ipmitool sensor list` (may need sudo)
- [ ] Look up the CUDA compatibility for your current driver version on NVIDIA's website

## Review Questions

1. Why does the driver version determine which CUDA versions you can use?
2. What's the difference between the GPU driver and GPU firmware?
3. What does the BMC do that you can't do from the OS?
4. A customer upgraded PyTorch and now gets "CUDA driver version insufficient." What do they need to upgrade?
5. Why do driver updates on a DGX require planning and coordination?

---

*Previous: [[M02L06 - NCCL In Depth]] | Next: [[M02L08 - DCGM and NVSM]]*
