# M01L07 - Pipes & Redirection

## The Big Idea

Pipes and redirection are what make the Linux command line truly powerful. Instead of using one tool at a time, you chain them together to do complex things in a single line.

## Output Redirection

### `>` — Send Output to a File (Overwrite)
```bash
# Save the output of a command to a file
$ echo "Hello, world!" > greeting.txt

# Save a directory listing
$ ls -la /var/log/ > log-listing.txt

# WARNING: This overwrites the file if it already exists!
```

### `>>` — Append to a File
```bash
# Add to the end of a file without overwriting
$ echo "Another line" >> greeting.txt

# Append today's GPU status to a log
$ nvidia-smi >> gpu-status-$(date +%Y-%m-%d).log
```

### `2>` — Redirect Errors
Commands have two output streams:
- **stdout** (standard output) — normal output → file descriptor 1
- **stderr** (standard error) — error messages → file descriptor 2

```bash
# Save errors to a file
$ grep -r "pattern" /var/log/ 2> errors.txt

# Save normal output AND errors to separate files
$ command > output.txt 2> errors.txt

# Save everything to one file
$ command > all-output.txt 2>&1
```

### Input Redirection
```bash
# Feed a file as input to a command
$ grep "error" < /var/log/syslog
```

## Pipes — The Real Power

The pipe (`|`) takes the output of one command and feeds it as input to the next.

```bash
command1 | command2 | command3
```

This is like an assembly line — each command does one thing and passes the result to the next.

### Essential Pipe Patterns

```bash
# Search logs for errors, then show only the last 20
$ grep "error" /var/log/syslog | tail -20

# Find all unique IP addresses in a log
$ grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" access.log | sort | uniq -c | sort -rn | head -10

# Count processes per user
$ ps aux | awk '{print $1}' | sort | uniq -c | sort -rn

# Find large log files
$ find /var/log -name "*.log" -exec ls -lh {} \; | sort -k5 -h

# Extract GPU temperatures from nvidia-smi
$ nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader
```

## Key Commands for Pipe Chains

| Command | What it does | Example |
|---------|-------------|---------|
| `grep` | Filter lines matching a pattern | `grep "error"` |
| `sort` | Sort lines alphabetically/numerically | `sort -n` (numeric), `sort -r` (reverse) |
| `uniq` | Remove duplicate adjacent lines | `uniq -c` (count duplicates) |
| `wc` | Count lines, words, characters | `wc -l` (lines only) |
| `head` | Keep first N lines | `head -20` |
| `tail` | Keep last N lines | `tail -20` |
| `awk` | Column-based processing | `awk '{print $2}'` (print 2nd column) |
| `sed` | Stream editing (find/replace) | `sed 's/old/new/g'` |
| `cut` | Extract columns by delimiter | `cut -d',' -f2` (2nd CSV column) |
| `tr` | Translate/delete characters | `tr 'A-Z' 'a-z'` (lowercase) |
| `tee` | Output to both screen AND file | `tee output.txt` |

## `tee` — Split the Stream

`tee` is like a T-junction in plumbing — it takes input and sends it to two places:

```bash
# Display output on screen AND save to file
$ nvidia-smi | tee gpu-status.txt

# Log a command's output while also seeing it
$ grep "error" /var/log/syslog | tee errors-found.txt | wc -l
```

## Command Substitution

Use `$(...)` to use a command's output as part of another command:

```bash
# Create a filename with today's date
$ echo "Report for $(date +%Y-%m-%d)" > report.txt

# Count files in a directory
$ echo "There are $(ls /var/log | wc -l) files in /var/log"

# Save GPU info with timestamp
$ nvidia-smi > "gpu-$(date +%Y%m%d-%H%M%S).log"
```

## Real-World Examples

```bash
# Find the 10 most common errors in a log
$ grep -i "error" /var/log/syslog | sort | uniq -c | sort -rn | head -10

# Find which GPUs have the highest memory usage
$ nvidia-smi --query-gpu=index,memory.used --format=csv,noheader,nounits | sort -t',' -k2 -rn

# Get a quick count of how many times each type of NCCL error appears
$ grep "NCCL" /var/log/syslog | grep -oP "NCCL \w+" | sort | uniq -c | sort -rn

# Find all Python processes and show their CPU/memory usage
$ ps aux | grep python | awk '{printf "PID: %s  CPU: %s%%  MEM: %s%%  CMD: %s\n", $2, $3, $4, $11}'

# Check if any filesystem is over 90% full
$ df -h | awk '{print $5}' | grep -v Use | sed 's/%//' | awk '$1 > 90 {print "WARNING: Filesystem over 90%!"}'
```

---

## Exercises

- [ ] Use `>` to save the output of `ls -la /var/log` to a file
- [ ] Use `>>` to append the output of `date` to that same file
- [ ] Use a pipe to count how many files are in `/var/log`: `ls /var/log | wc -l`
- [ ] Chain `grep`, `sort`, and `uniq -c` to find unique lines in a file
- [ ] Use `tee` to display a command's output on screen AND save it to a file
- [ ] Use command substitution to create a file with today's date in the name
- [ ] Build a one-liner: find the 5 most common words in a log file

## Review Questions

1. What's the difference between `>` and `>>`?
2. What does `2>&1` do?
3. What does `grep "error" log.txt | wc -l` tell you?
4. How would you save `nvidia-smi` output to a file AND see it on screen at the same time?
5. Explain what this command does step by step: `cat access.log | grep "404" | awk '{print $7}' | sort | uniq -c | sort -rn | head -10`

---

*Previous: [[M01L06 - Processes & Monitoring]] | Next: [[M01L08 - Environment Variables & PATH]]*
