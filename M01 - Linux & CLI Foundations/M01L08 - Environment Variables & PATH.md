# M01L08 - Environment Variables & PATH

## The Big Idea

Environment variables are settings that programs can read. They control how your shell behaves, where it looks for programs, and how other tools configure themselves. Understanding these is essential for installing software and troubleshooting.

## What Are Environment Variables?

Think of them as system-wide or session-wide preferences. A name paired with a value.

```bash
# View all environment variables
$ env

# View a specific variable
$ echo $HOME
/home/cynthia

$ echo $USER
cynthia

$ echo $SHELL
/bin/bash
```

The `$` sign means "give me the value of this variable."

## Common Environment Variables

| Variable | What it holds | Example |
|----------|--------------|---------|
| `HOME` | Your home directory | `/home/cynthia` |
| `USER` | Your username | `cynthia` |
| `SHELL` | Your default shell | `/bin/bash` |
| `PATH` | Where to look for programs | `/usr/local/bin:/usr/bin:/bin` |
| `PWD` | Your current directory | `/var/log` |
| `HOSTNAME` | The machine's name | `dgx-worker-01` |
| `LANG` | Language/locale setting | `en_US.UTF-8` |
| `CUDA_HOME` | Where CUDA is installed | `/usr/local/cuda` |
| `LD_LIBRARY_PATH` | Where to find shared libraries | `/usr/local/cuda/lib64` |

## Setting Variables

### Temporary (current session only)
```bash
# Set a variable
$ MY_VAR="hello"

# Use it
$ echo $MY_VAR
hello

# Export it so child processes can see it too
$ export MY_VAR="hello"
```

**`export` is important** — without it, the variable is only available in your current shell, not in programs you run from that shell.

### Using variables in commands
```bash
# Use in a path
$ cd $HOME/Documents

# Use in a string
$ echo "Hello, $USER"
Hello, cynthia

# Use in a filename
$ cp log.txt "$HOME/logs/log-$(date +%F).txt"
```

## PATH — The Most Important Variable

`PATH` tells your shell where to look for executable programs. When you type `python3`, the shell searches through every directory in `PATH` until it finds a program called `python3`.

```bash
# See your current PATH
$ echo $PATH
/usr/local/cuda/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

Each directory is separated by `:`. The shell searches them **left to right** and uses the first match.

### Adding to PATH

```bash
# Add CUDA tools to your PATH (temporary)
$ export PATH=/usr/local/cuda/bin:$PATH

# Add a custom scripts directory
$ export PATH=$HOME/scripts:$PATH
```

Note the pattern: `export PATH=/new/dir:$PATH` — this prepends the new directory so it's searched first.

### "Command not found" — Usually a PATH Problem

```bash
$ nvcc
bash: nvcc: command not found
```

This usually means either:
1. The program isn't installed
2. The program is installed but its directory isn't in your `PATH`

```bash
# Find where a program actually lives
$ which python3
/usr/bin/python3

# Or search more broadly
$ find / -name "nvcc" 2>/dev/null
/usr/local/cuda/bin/nvcc

# Then add that directory to PATH
$ export PATH=/usr/local/cuda/bin:$PATH
$ nvcc --version
# Now it works!
```

## Making Variables Persistent

Variables set with `export` disappear when you close your terminal. To make them permanent, add them to your shell's config file.

### `.bashrc` — Your Shell Configuration

Every time you open a new bash shell, it reads `~/.bashrc`. Add your custom settings there:

```bash
# Edit your bashrc
$ nano ~/.bashrc

# Add these lines at the bottom:
export CUDA_HOME=/usr/local/cuda
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH

# Save and exit (Ctrl+O, Enter, Ctrl+X in nano)

# Apply the changes immediately
$ source ~/.bashrc
```

`source` reads and executes the file in your current session — you don't need to open a new terminal.

## Aliases — Shortcuts for Common Commands

Add these to `~/.bashrc` too:

```bash
alias ll='ls -la'
alias gs='git status'
alias gpu='nvidia-smi'
alias gpul='watch -n 2 nvidia-smi'
alias ports='netstat -tulanp'
alias logs='cd /var/log'
```

---

## Exercises

- [ ] Run `env` and look through the output. Identify variables you recognize.
- [ ] Set a custom variable: `export MY_TEST="hello world"` then `echo $MY_TEST`
- [ ] Run `echo $PATH` and identify each directory listed
- [ ] Use `which` to find where `ls`, `grep`, and `python3` live
- [ ] Create an alias in your current session and test it
- [ ] Add a useful alias to your `~/.bashrc` and reload with `source ~/.bashrc`

## Review Questions

1. What's the difference between `MY_VAR="hello"` and `export MY_VAR="hello"`?
2. You type `nvcc` and get "command not found" but you know CUDA is installed at `/usr/local/cuda/`. What do you do?
3. How do you make an environment variable persist across terminal sessions?
4. What does `source ~/.bashrc` do and why do you need it?
5. What's the significance of the order of directories in `$PATH`?

---

*Previous: [[M01L07 - Pipes & Redirection]] | Next: [[M01L09 - SSH & Remote Access]]*
