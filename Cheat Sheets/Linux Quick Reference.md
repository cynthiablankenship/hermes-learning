# 🛠️ Linux Command Cheat Sheet

Quick reference for the most commonly used commands.

## Navigation
| Command | What it does |
|---------|-------------|
| `pwd` | Show current directory |
| `ls` | List files |
| `ls -la` | List all files with details |
| `cd /path` | Change to directory |
| `cd ~` | Go to home directory |
| `cd ..` | Go up one level |
| `cd -` | Go to previous directory |

## Files & Directories
| Command | What it does |
|---------|-------------|
| `touch file.txt` | Create empty file |
| `mkdir dir` | Create directory |
| `mkdir -p path/to/dir` | Create nested directories |
| `cp src dest` | Copy file |
| `cp -r src dest` | Copy directory recursively |
| `mv old new` | Move or rename |
| `rm file` | Delete file |
| `rm -r dir` | Delete directory |
| `rm -ri dir` | Delete with confirmation |

## Reading & Searching
| Command | What it does |
|---------|-------------|
| `cat file` | Print entire file |
| `less file` | Page through file |
| `head -n 20 file` | First 20 lines |
| `tail -n 20 file` | Last 20 lines |
| `tail -f file` | Follow file in real-time |
| `grep "pattern" file` | Search for text |
| `grep -i "pattern" file` | Case-insensitive search |
| `grep -r "pattern" dir/` | Recursive search |
| `grep -c "pattern" file` | Count matches |
| `grep -n "pattern" file` | Show line numbers |
| `grep -B2 -A2 "pat" file` | Show context |

## Permissions
| Command | What it does |
|---------|-------------|
| `chmod 755 script.sh` | rwxr-xr-x |
| `chmod 644 file.txt` | rw-r--r-- |
| `chmod 600 key.pem` | rw------- |
| `chmod +x script.sh` | Make executable |
| `chown user:group file` | Change ownership |

## Processes
| Command | What it does |
|---------|-------------|
| `ps aux` | List all processes |
| `top` | Live process monitor |
| `htop` | Better process monitor |
| `kill PID` | End process gracefully |
| `kill -9 PID` | Force kill process |
| `nvidia-smi` | GPU status |
| `df -h` | Disk usage |
| `free -h` | Memory usage |

## Pipes & Redirection
| Symbol | What it does |
|--------|-------------|
| `> file` | Write output to file (overwrite) |
| `>> file` | Append output to file |
| `2> file` | Redirect errors to file |
| `\|` | Pipe output to next command |
| `tee file` | Output to screen AND file |

## SSH
| Command | What it does |
|---------|-------------|
| `ssh user@host` | Connect to remote |
| `ssh -i key.pem user@host` | Connect with specific key |
| `scp file user@host:/path` | Copy file to remote |
| `ssh-keygen -t ed25519` | Generate SSH key |

## Useful One-Liners
```bash
# Find the 10 most common log entries
grep -i "error" logfile | sort | uniq -c | sort -rn | head -10

# Watch GPU usage continuously
watch -n 2 nvidia-smi

# Find large files
find / -size +100M -exec ls -lh {} \; 2>/dev/null

# Count files in directory
ls | wc -l

# System report one-liner
echo "Host: $(hostname) | Up: $(uptime -p) | Disk: $(df -h / | tail -1 | awk '{print $5}') used"
```
