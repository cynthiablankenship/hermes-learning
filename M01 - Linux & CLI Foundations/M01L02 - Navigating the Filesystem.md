# M01L02 - Navigating the Filesystem

## The Big Idea

Everything in Linux is a file. Directories (folders) are just files that contain other files. You navigate using three main commands: `pwd`, `ls`, and `cd`.

## Essential Commands

### `pwd` — Print Working Directory
Tells you where you are right now.

```bash
$ pwd
/home/cynthia
```

Use this whenever you feel lost. It's your "where am I?" command.

### `ls` — List Directory Contents
Shows you what's in the current directory.

```bash
$ ls
Desktop  Documents  Downloads  Pictures
```

Useful variations:
| Command | What it does |
|---------|-------------|
| `ls` | List files in current directory |
| `ls -l` | Long format (shows permissions, size, date) |
| `ls -la` | Long format + hidden files (starts with `.`) |
| `ls -lh` | Long format with human-readable sizes (KB, MB, GB) |
| `ls /var/log` | List contents of `/var/log` without going there |
| `ls -lt` | Sort by modification time (newest first) |

**`ls -la` is one of the most useful commands you'll ever use.** Memorize it.

### `cd` — Change Directory
Moves you to a different directory.

```bash
$ cd Documents        # Go into the Documents folder
$ cd /var/log         # Go directly to /var/log (absolute path)
$ cd ..               # Go up one level
$ cd ../..            # Go up two levels
$ cd ~                # Go to your home directory
$ cd -                # Go back to where you just were
```

## Absolute vs Relative Paths

This is a critical concept.

**Absolute path** — starts from `/`, the root. Always works no matter where you are.
```bash
cd /var/log/nvidia/   # Always goes to the same place
```

**Relative path** — starts from where you are right now.
```bash
cd Documents/          # Goes into Documents inside your current directory
```

Analogy:
- **Absolute path** is like a full mailing address: "123 Main St, Memphis, TN 38101"
- **Relative path** is like saying "go to the kitchen" — it depends on where you are now

## Special Directory Symbols

| Symbol | Meaning |
|--------|---------|
| `/` | Root directory (top of everything) |
| `~` | Your home directory (`/home/cynthia`) |
| `.` | Current directory |
| `..` | Parent directory (one level up) |
| `-` | Previous directory you were in (with `cd -`) |

## Practical Example: Finding Logs on a DGX

```bash
# Where am I?
$ pwd
/home/cynthia

# Go to where logs live
$ cd /var/log

# What's here?
$ ls
auth.log  dmesg  nvidia/  syslog  kern.log

# Go into the nvidia log directory
$ cd nvidia

# Show me the files with details
$ ls -lh
total 8.0K
-rw-r--r-- 1 root root 2.3K Apr  9 10:00 nvsm.log
-rw-r--r-- 1 root root 1.1K Apr  9 09:45 gpu-monitor.log

# Go back to where I was
$ cd -
/home/cynthia
```

## Tab Completion

**This will save you enormous amounts of time.** When typing a path or command, press **Tab** and Linux will auto-complete it for you.

```bash
$ cd /var/lo[Tab]     # Completes to /var/log/
$ cd /var/log/nvi[Tab] # Completes to /var/log/nvidia/
```

If there are multiple options, press Tab twice to see them all. **Use this constantly.** It also prevents typos in paths.

---

## Exercises

- [ ] Run `pwd` and note where you are
- [ ] Run `ls`, then `ls -l`, then `ls -la`. Notice the differences.
- [ ] Navigate to `/var/log` using `cd`. What files do you see?
- [ ] Navigate back to your home directory three different ways (`cd ~`, `cd /home/cynthia`, `cd`)
- [ ] Use tab completion to navigate to `/var/log` without typing the full path
- [ ] Run `ls -la ~` and `ls -la /`. Compare the outputs.

## Review Questions

1. What's the difference between `cd /home/cynthia` and `cd home/cynthia`?
2. What does the `-a` flag do in `ls -la`?
3. You're in `/var/log/nvidia/` and want to go to `/var/log/`. What command do you use (shortest option)?
4. What does `cd -` do?
5. You type `cd /var/lo` and press Tab. What happens? What if you press Tab again?

---

*Previous: [[M01L01 - What is Linux]] | Next: [[M01L03 - File Operations]]*
