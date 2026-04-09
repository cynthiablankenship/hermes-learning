# M01L06 - Processes & Monitoring

## The Big Idea

Every program running on a Linux system is a **process.** When troubleshooting, you need to see what's running, how much CPU/memory it's using, and sometimes kill misbehaving processes.

## Viewing Processes

### `ps` — Process Snapshot
```bash
# Show processes in your current terminal session
$ ps

# Show ALL processes on the system with full details
$ ps aux

# Show processes in a tree (parent-child relationships)
$ ps auxf

# Find a specific process by name
$ ps aux | grep docker
```

The `ps aux` output columns:
```
USER  PID  %CPU  %MEM  VSZ   RSS   TTY  STAT  START  TIME  COMMAND
root  1    0.0   0.1   16936 9480  ?    Ss    Apr08  0:03  /sbin/init
```

| Column | Meaning |
|--------|---------|
| USER | Who owns the process |
| PID | Process ID (unique number) |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| STAT | Process state (R=running, S=sleeping, Z=zombie) |
| COMMAND | The actual command that started it |

### `top` — Live Process Monitor
```bash
$ top
```

Shows a continuously updating view of processes, sorted by CPU usage. Like Task Manager.

Key shortcuts inside `top`:
- `M` — sort by memory usage
- `P` — sort by CPU usage
- `q` — quit
- `k` — kill a process (prompts for PID)

### `htop` — Better Live Monitor (if installed)
```bash
$ htop
```

Like `top` but with colors, mouse support, and a much better interface. May need to be installed: `sudo apt install htop`.

## GPU Monitoring

On DGX/HGX systems, you also need to monitor GPUs:

```bash
# NVIDIA System Management Interface — shows GPU status
$ nvidia-smi

# Watch it update every 2 seconds
$ watch -n 2 nvidia-smi

# Show only GPU utilization
$ nvidia-smi --query-gpu=utilization.gpu,memory.used,memory.total --format=csv
```

`nvidia-smi` output shows:
- GPU index (0, 1, 2, ...)
- GPU name (A100, H100, etc.)
- Temperature
- Memory usage
- GPU utilization %
- Processes using the GPU

## Killing Processes

```bash
# Kill by PID (graceful)
$ kill 12345

# Kill by PID (force — use if regular kill doesn't work)
$ kill -9 12345

# Kill by name
$ killall python3

# Kill using pkill (supports partial name matching)
$ pkill -f "training_script"
```

Kill signals:
| Signal | Flag | Meaning |
|--------|------|---------|
| SIGTERM | `-15` (default) | "Please shut down gracefully" |
| SIGKILL | `-9` | "Die immediately, no cleanup" — use as last resort |

**Always try regular `kill` first.** Only use `kill -9` if the process refuses to die.

## Checking System Resources

```bash
# Disk usage
$ df -h                    # Show disk space for all mounted filesystems
$ du -sh /var/log/         # Show total size of a directory

# Memory usage
$ free -h                  # Show RAM usage

# Who's logged in
$ who                      # List logged-in users
$ w                        # List logged-in users + what they're doing
```

## Systemd — Managing Services

Modern Linux uses `systemd` to manage long-running services (daemons):

```bash
# Check if a service is running
$ sudo systemctl status docker

# Start a service
$ sudo systemctl start docker

# Stop a service
$ sudo systemctl stop docker

# Restart a service
$ sudo systemctl restart docker

# Enable a service to start on boot
$ sudo systemctl enable docker

# View service logs
$ sudo journalctl -u docker -n 50    # Last 50 lines
$ sudo journalctl -u docker -f       # Follow in real-time
```

## Practical Example: Debugging a Slow DGX

```bash
# Step 1: Check overall system load
$ top    # Is CPU pegged? By what?

# Step 2: Check GPU status
$ nvidia-smi    # Are GPUs being used? By whom?

# Step 3: Check disk space
$ df -h    # Is any filesystem full?

# Step 4: Check memory
$ free -h    # Is RAM exhausted?

# Step 5: Check specific service
$ sudo systemctl status docker    # Is Docker running properly?

# Step 6: Check recent logs
$ sudo journalctl -u docker --since "1 hour ago"
```

---

## Exercises

- [ ] Run `ps aux` and identify the process with the highest PID
- [ ] Run `top` and sort by memory usage (press M)
- [ ] Check GPU status with `nvidia-smi` (if on a system with GPUs)
- [ ] Run `df -h` and identify which filesystem has the most free space
- [ ] Run `free -h` and understand the output
- [ ] Check the status of the SSH service: `sudo systemctl status sshd`
- [ ] Use `ps aux | grep` to find a specific running process

## Review Questions

1. What's the difference between `kill 1234` and `kill -9 1234`?
2. How do you watch `nvidia-smi` update continuously?
3. What does `ps aux` show you that `ps` alone doesn't?
4. You see a process consuming 99% CPU. What steps would you take?
5. What command shows you disk space usage in human-readable format?

---

*Previous: [[M01L05 - Permissions & Ownership]] | Next: [[M01L07 - Pipes & Redirection]]*
