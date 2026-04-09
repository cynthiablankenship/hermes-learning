# M01L04 - Reading & Searching Files

## Why This Lesson Matters Most

This is **the most important lesson in Module 1 for your job.** You read logs for a living. These commands are how you do it efficiently on Linux.

## Reading Files

### `cat` — Print Entire File
Short for "concatenate." Dumps the entire file to your screen.

```bash
$ cat /var/log/syslog
```

Good for short files. Bad for long ones — it'll blast past your screen.

### `less` — Page Through a File
This is your best friend for reading long files.

```bash
$ less /var/log/syslog
```

Navigation inside `less`:
| Key | Action |
|-----|--------|
| Space / Page Down | Forward one page |
| b / Page Up | Back one page |
| Down arrow / Enter | Forward one line |
| Up arrow | Back one line |
| `/pattern` | Search forward for "pattern" |
| `?pattern` | Search backward for "pattern" |
| n | Jump to next search match |
| N | Jump to previous search match |
| g | Go to beginning of file |
| G | Go to end of file |
| q | Quit |
| F | Follow mode (like `tail -f`) |

**Pro tip:** Start `less` with `-S` to disable line wrapping — long lines scroll horizontally instead of wrapping:
```bash
$ less -S /var/log/syslog
```

### `head` — Show the Beginning
```bash
# First 10 lines (default)
$ head /var/log/syslog

# First 50 lines
$ head -n 50 /var/log/syslog
```

### `tail` — Show the End
This is **incredibly useful for logs.**

```bash
# Last 10 lines (default)
$ tail /var/log/syslog

# Last 100 lines
$ tail -n 100 /var/log/syslog

# FOLLOW MODE — watch the file grow in real-time
$ tail -f /var/log/nvidia/gpu-monitor.log
```

`tail -f` is probably the command you'll use most at your job. It stays open and shows new lines as they're written. Essential for watching logs in real-time while troubleshooting.

## Searching with `grep`

`grep` searches for patterns inside files. This is your primary tool for finding things in logs.

### Basic Usage
```bash
# Find all lines containing "error" (case-sensitive)
$ grep "error" /var/log/syslog

# Case-insensitive search
$ grep -i "error" /var/log/syslog

# Show line numbers
$ grep -n "error" /var/log/syslog

# Invert match — show lines that DON'T contain "error"
$ grep -v "error" /var/log/syslog
```

### Searching Multiple Files
```bash
# Search all .log files in a directory
$ grep "cudaError" /var/log/nvidia/*.log

# Recursive search through all subdirectories
$ grep -r "NCCL" /var/log/

# Just show filenames that match (don't show the content)
$ grep -l "GPU" /var/log/*.log
```

### Counting Matches
```bash
# How many lines contain "error"?
$ grep -c "error" /var/log/syslog
```

### Context — Show Surrounding Lines
This is **huge** for log analysis. You don't just want the error line — you want to see what happened before and after.

```bash
# Show 3 lines before each match
$ grep -B 3 "error" /var/log/syslog

# Show 3 lines after each match
$ grep -A 3 "error" /var/log/syslog

# Show 3 lines before AND after
$ grep -B 3 -A 3 "error" /var/log/syslog
```

### Useful grep patterns for your job
```bash
# Find CUDA errors
$ grep -i "cuda.*error" /var/log/syslog

# Find memory-related issues
$ grep -i "out of memory" /var/log/syslog

# Find NCCL timeouts (common GPU communication issue)
$ grep "NCCL.*timeout" /var/log/syslog

# Find failed service starts
$ grep -i "failed to start" /var/log/syslog
```

## Combining Commands (Preview of Lesson 7)

You can chain commands with pipes (`|`) to get powerful results:

```bash
# Find errors, then show only the last 10
$ grep "error" /var/log/syslog | tail -10

# Count how many errors came from GPU 0
$ grep "GPU 0" /var/log/syslog | grep -c "error"

# Find errors and sort them by uniqueness
$ grep "error" /var/log/syslog | sort | uniq -c | sort -rn | head -20
```

---

## Exercises

- [ ] Use `cat` to read a short file (like `/etc/hostname`)
- [ ] Use `less` to page through a longer file. Practice searching with `/`
- [ ] Use `head -n 5` and `tail -n 5` on the same file. Compare what you see
- [ ] Use `grep` to search for a word in a log file
- [ ] Use `grep -i` to do a case-insensitive search
- [ ] Use `grep -n` to see line numbers with your matches
- [ ] Use `grep -B 2 -A 2` to see context around a match
- [ ] Use `grep -c` to count how many times something appears in a file

## Review Questions

1. What's the difference between `cat` and `less`? When would you use each?
2. How do you watch a log file in real-time as it grows?
3. What does `grep -v "debug" logfile` do?
4. You want to see what happened right before and after a CUDA error. What command do you use?
5. What does this command do: `grep -i "error" /var/log/syslog | wc -l`?

---

*Previous: [[M01L03 - File Operations]] | Next: [[M01L05 - Permissions & Ownership]]*
