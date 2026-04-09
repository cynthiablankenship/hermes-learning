# M01L03 - File Operations

## The Big Idea

You need to create, copy, move, rename, and delete files and directories. These are the building blocks of everything else you'll do.

## Essential Commands

### Creating Files and Directories

```bash
# Create an empty file
$ touch report.txt

# Create multiple files at once
$ touch log1.txt log2.txt log3.txt

# Create a directory
$ mkdir projects

# Create nested directories (the -p flag creates parent dirs too)
$ mkdir -p projects/nvidia/dgx-logs
```

**`mkdir -p` is important** — without `-p`, the command fails if the parent directory doesn't exist. With `-p`, it creates the whole path.

### Copying

```bash
# Copy a file
$ cp report.txt report-backup.txt

# Copy a file to a different directory
$ cp report.txt projects/

# Copy a directory and everything in it (-r = recursive)
$ cp -r projects/ projects-backup/
```

### Moving and Renaming

In Linux, **moving and renaming are the same command** (`mv`). There is no separate `rename`.

```bash
# Rename a file
$ mv report.txt final-report.txt

# Move a file to a different directory
$ mv final-report.txt projects/

# Move and rename at the same time
$ mv old-log.txt /var/log/nvidia/new-log.txt
```

### Deleting

⚠️ **Be careful with delete commands.** There is no Recycle Bin in Linux. When it's gone, it's gone.

```bash
# Delete a file
$ rm unwanted-file.txt

# Delete a directory and everything in it
$ rm -r projects/

# Interactive mode — asks before each deletion (safer)
$ rm -i report.txt
rm: remove regular empty file 'report.txt'? y

# THE NUCLEAR OPTION — DO NOT RUN THIS
$ rm -rf /       # This would delete everything on the system
```

**Safety tip:** Add this to your `.bashrc` to make `rm` always ask for confirmation:
```bash
alias rm='rm -i'
```

## Understanding `ls -l` Output

When you run `ls -l`, you get output like this:
```
-rw-r--r-- 1 cynthia users 4096 Apr  9 10:00 report.txt
drwxr-xr-x 2 cynthia users 4096 Apr  9 10:05 projects
```

Each column means something:

```
-rw-r--r--  1  cynthia  users  4096  Apr  9 10:00  report.txt
│          │  │        │      │     │               │
│          │  │        │      │     │               └── File name
│          │  │        │      │     └── Modification date/time
│          │  │        │      └── File size (bytes)
│          │  │        └── Group owner
│          │  └── User owner
│          └── Number of hard links
└── Permissions (covered in Lesson 5)
```

The first character tells you the type:
- `-` = regular file
- `d` = directory
- `l` = symbolic link (shortcut)

## Wildcards (Glob Patterns)

Wildcards let you work with multiple files at once:

| Pattern | Matches |
|---------|---------|
| `*` | Any characters |
| `*.txt` | All files ending in .txt |
| `log*` | All files starting with "log" |
| `?.txt` | Any single character + .txt (a.txt, b.txt) |

```bash
# Copy all .log files
$ cp *.log /backup/

# Delete all .tmp files
$ rm *.tmp

# List all files with "nvidia" in the name
$ ls *nvidia*
```

## Practical Example: Organizing Log Files

```bash
# Create a directory for today's logs
$ mkdir -p ~/logs/2026-04-09

# Copy relevant logs there
$ cp /var/log/nvidia/*.log ~/logs/2026-04-09/

# Rename a log to note what it's for
$ mv ~/logs/2026-04-09/gpu-monitor.log ~/logs/2026-04-09/gpu-monitor-before-fix.log

# Check what you've got
$ ls -lh ~/logs/2026-04-09/
```

---

## Exercises

- [ ] Create a directory called `practice` in your home directory
- [ ] Inside `practice`, create three files: `file1.txt`, `file2.txt`, `file3.txt`
- [ ] Create a subdirectory called `backup` inside `practice`
- [ ] Copy all three `.txt` files into `backup`
- [ ] Rename `file1.txt` (the original, not the copy) to `renamed-file.txt`
- [ ] Delete `file2.txt` from the original location
- [ ] Run `ls -la` in both `practice` and `practice/backup` to verify everything looks right
- [ ] Delete the entire `practice` directory when done

## Review Questions

1. What's the difference between `cp file.txt backup/` and `cp file.txt backup`? (Hint: does `backup` exist as a directory?)
2. How would you create the directory path `/home/cynthia/projects/dgx/logs` in one command, even if `projects` doesn't exist yet?
3. What does `mv old.txt new.txt` do vs `cp old.txt new.txt`?
4. Why is `rm -rf` dangerous? What precautions can you take?
5. What does `cp *.log /tmp/` do?

---

*Previous: [[M01L02 - Navigating the Filesystem]] | Next: [[M01L04 - Reading & Searching Files]]*
