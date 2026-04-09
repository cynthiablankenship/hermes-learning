# M01L10 - Lab: Navigating a DGX via SSH

## Lab Overview

This lab ties together everything from Module 1. You'll simulate the workflow of connecting to a DGX system and performing common troubleshooting tasks.

**Prerequisites:** Complete lessons 1.1 through 1.9 before attempting this lab.

**What you need:**
- Access to a Linux system (DGX, VM, WSL, or any Linux machine you can SSH into)
- Your terminal/SSH client

---

## Part 1: Connect and Orient

Connect to your system and figure out what you're working with.

```bash
# SSH into your system
$ ssh your-username@your-server

# Task 1: What's the hostname?
$ hostname

# Task 2: What OS is this running?
$ cat /etc/os-release

# Task 3: What kernel version?
$ uname -r

# Task 4: How many CPUs does this system have?
$ nproc

# Task 5: How much RAM?
$ free -h
```

- [ ] Part 1 complete

---

## Part 2: Explore the Filesystem

Navigate around and understand the system layout.

```bash
# Task 6: What's in /opt/? (Look for NVIDIA software)
$ ls -la /opt/

# Task 7: What log files exist?
$ ls -la /var/log/

# Task 8: Is there an NVIDIA-specific log directory?
$ ls -la /var/log/nvidia/   # May not exist on non-DGX systems

# Task 9: What's the full path to the CUDA installation?
$ ls -la /usr/local/cuda*   # Common CUDA location
$ which nvcc                # Or find it via PATH

# Task 10: What's in the NVIDIA device directory?
$ ls -la /dev/nvidia*       # GPU device files
```

- [ ] Part 2 complete

---

## Part 3: System Health Check

Check the overall health of the system.

```bash
# Task 11: Check disk space. Is anything over 80%?
$ df -h

# Task 12: Check memory usage
$ free -h

# Task 13: What are the top 5 processes by CPU?
$ ps aux --sort=-%cpu | head -6

# Task 14: What are the top 5 processes by memory?
$ ps aux --sort=-%mem | head -6

# Task 15: How long has this system been up?
$ uptime
```

- [ ] Part 3 complete

---

## Part 4: GPU Check

If you're on a system with NVIDIA GPUs:

```bash
# Task 16: Show full GPU status
$ nvidia-smi

# Task 17: What GPU model(s) are installed?
$ nvidia-smi --query-gpu=name --format=csv,noheader

# Task 18: What's the temperature of each GPU?
$ nvidia-smi --query-gpu=index,temperature.gpu --format=csv

# Task 19: How much memory is used/total on each GPU?
$ nvidia-smi --query-gpu=index,memory.used,memory.total --format=csv

# Task 20: Are any processes using GPUs?
$ nvidia-smi --query-compute-apps=pid,gpu_uuid,used_memory --format=csv
```

- [ ] Part 4 complete

---

## Part 5: Log Investigation

Practice reading and searching logs — your core job skill.

```bash
# Task 21: How many lines are in syslog?
$ wc -l /var/log/syslog

# Task 22: Show the last 20 lines of syslog
$ tail -20 /var/log/syslog

# Task 23: Search for any error messages
$ grep -i "error" /var/log/syslog | tail -20

# Task 24: Count how many error messages exist
$ grep -ic "error" /var/log/syslog

# Task 25: Find the 5 most common error messages
$ grep -i "error" /var/log/syslog | sort | uniq -c | sort -rn | head -5

# Task 26: Search for NVIDIA/CUDA related messages
$ grep -i "nvidia\|cuda" /var/log/syslog | tail -20

# Task 27: Save today's GPU status to a dated file
$ nvidia-smi > "$HOME/gpu-status-$(date +%Y-%m-%d).txt"

# Task 28: Find all .log files larger than 10MB
$ find /var/log -name "*.log" -size +10M -exec ls -lh {} \;
```

- [ ] Part 5 complete

---

## Part 6: Cleanup and Report

```bash
# Task 29: Create a summary report
$ cat << 'EOF' > ~/system-report-$(date +%Y-%m-%d).txt
=== System Report ===
Date: $(date)
Hostname: $(hostname)
OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)
Kernel: $(uname -r)
Uptime: $(uptime)
Disk Usage:
$(df -h)
Memory:
$(free -h)
GPUs:
$(nvidia-smi)
EOF

# Task 30: Verify the report was created
$ ls -lh ~/system-report-*.txt
$ cat ~/system-report-*.txt
```

- [ ] Part 6 complete

---

## Lab Completion

- [ ] All 30 tasks completed
- [ ] Comfortable navigating the filesystem
- [ ] Comfortable reading and searching logs
- [ ] Comfortable checking system and GPU health
- [ ] Can create and save a basic system report

**Congratulations!** You've completed Module 1. You can now navigate a Linux system, read logs, check system health, and perform basic troubleshooting — the foundation for everything that comes next.

---

*Next up: [[M02 - Containers & Docker]]*
