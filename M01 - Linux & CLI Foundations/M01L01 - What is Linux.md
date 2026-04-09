# M01L01 - What is Linux?

## The Short Version

Linux is an operating system — like Windows or macOS — but it's free, open-source, and runs almost everything behind the scenes. Most websites you visit, most cloud services, and essentially **all** AI infrastructure runs on Linux.

## Why Linux Matters for AI

- **DGX and HGX systems run Linux** — specifically Ubuntu or RHEL (Red Hat Enterprise Linux)
- **Docker containers are Linux-based** — even when you run Docker on Windows, it's running a Linux VM under the hood
- **AI frameworks (PyTorch, TensorFlow) are designed for Linux first** — Windows support is an afterthought
- **Log files live on Linux systems** — when you're reading logs at Dell, you're reading Linux logs
- **SSH access** — when you connect to a customer's DGX, you're SSH-ing into a Linux machine

## How Linux Differs from Windows

| Windows | Linux |
|---------|-------|
| C:\Users\Cynthia\Documents | /home/cynthia/Documents |
| File Explorer | `ls`, `cd`, `pwd` (terminal commands) |
| .exe, .msi files | Packages via `apt` or `yum` |
| Drives (C:, D:) | Everything is under `/` (the root) |
| Task Manager | `top`, `htop`, `ps` |
| Notepad | `nano`, `vim`, `cat` |
| Hidden files: checkbox in settings | Hidden files start with `.` (like `.bashrc`) |

## The Linux Filesystem

Linux doesn't have drive letters. Everything starts at `/` (called "root"). Here's the important stuff:

```
/              ← The root of everything
├── home/      ← User directories (like C:\Users)
│   └── cynthia/  ← Your personal folder
├── etc/       ← System configuration files
├── var/       ← Variable data — THIS IS WHERE LOGS LIVE
│   └── log/   ← System log files
├── tmp/       ← Temporary files (cleared on reboot)
├── usr/       ← User-installed programs
│   ├── bin/   ← Executable programs
│   └── local/ ← Locally compiled software
├── opt/       ← Optional/third-party software (NVIDIA stuff often goes here)
├── proc/      ← Virtual filesystem — info about running processes
└── dev/       ← Device files (GPUs show up here)
```

**Key takeaway for your job:** Logs are usually in `/var/log/`. Configuration files are in `/etc/`. NVIDIA software is often in `/opt/` or `/usr/local/`.

## What is a Shell?

A shell is the program that takes your typed commands and executes them. The most common one is **bash** (Bourne Again Shell). When you open a terminal or SSH into a server, you're in a bash shell.

The **prompt** (the text before your cursor) usually looks something like:
```
cynthia@dgx01:~$
```

This tells you:
- `cynthia` — your username
- `dgx01` — the hostname (the server's name)
- `~` — your current directory (`~` is shorthand for your home directory)
- `$` — you're a normal user (`#` would mean you're root/admin)

## Distributions (Distros)

Linux comes in different "flavors" called distributions. For your work, you mainly need to know two:

- **Ubuntu** — Most common for DGX systems. Uses `apt` package manager.
- **RHEL (Red Hat Enterprise Linux)** — Used in some enterprise setups. Uses `yum` or `dnf` package manager.

The commands are 95% the same across distributions. Don't worry about the differences yet.

---

## Exercises

- [ ] Open a terminal (or WSL on Windows, or SSH into any Linux machine)
- [ ] Type `pwd` — what does it show? This is your current directory.
- [ ] Type `ls` — what do you see?
- [ ] Type `ls /` — this lists the root directory. Can you see the folders mentioned above?
- [ ] Type `uname -a` — this shows system information about the Linux machine you're on

## Review Questions

1. Where do log files typically live on a Linux system?
2. What's the difference between `/home/cynthia/Documents/report.txt` and `~/Documents/report.txt`?
3. If your prompt shows `root@server:/var/log#`, what does each part tell you?
4. Why doesn't Linux use drive letters like Windows?
5. Name two places you'd look for NVIDIA software on a Linux system.

---

*Next: [[M01L02 - Navigating the Filesystem]]*
