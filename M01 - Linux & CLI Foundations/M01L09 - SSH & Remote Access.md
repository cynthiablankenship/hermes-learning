# M01L09 - SSH & Remote Access

## The Big Idea

SSH (Secure Shell) is how you connect to remote servers. At Dell, when you need to read logs on a customer's DGX or access a test system, you SSH into it. This is your primary way of interacting with AI infrastructure.

## Basic SSH Connection

```bash
# Connect to a remote server
$ ssh cynthia@192.168.1.100

# Connect with a specific username
$ ssh admin@dgx-worker-01.example.com

# Connect with a specific SSH key
$ ssh -i ~/.ssh/my-key.pem cynthia@192.168.1.100

# Connect on a non-standard port
$ ssh -p 2222 cynthia@192.168.1.100
```

The format is: `ssh [options] username@hostname`

`hostname` can be:
- An IP address: `192.168.1.100`
- A domain name: `dgx01.company.com`
- A hostname on your network: `dgx-worker-01`

## First Connection — The Fingerprint Prompt

The first time you connect to a server, you'll see:

```
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
ED25519 key fingerprint is SHA256:AbCdEf123456...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes`. This saves the server's fingerprint so your computer can verify it in the future. If you see this prompt again for a server you've already connected to, **be cautious** — it might mean the server was reinstalled or someone is trying to intercept your connection (man-in-the-middle).

## SSH Keys — Password-Less Login

SSH keys are a pair of files: a **private key** (keep secret) and a **public key** (share with servers).

### Generate a Key Pair

```bash
$ ssh-keygen -t ed25519 -C "cynthia@company.com"
```

This creates:
- `~/.ssh/id_ed25519` — your private key (NEVER share this)
- `~/.ssh/id_ed25519.pub` — your public key (safe to share)

Press Enter to accept the default location. Optionally set a passphrase for extra security.

### Copy Your Public Key to a Server

```bash
# Automatic method (if available)
$ ssh-copy-id cynthia@192.168.1.100

# Manual method
$ cat ~/.ssh/id_ed25519.pub | ssh cynthia@192.168.1.100 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

After this, you can SSH to that server without a password — it authenticates using your key pair.

## SSH Config File — Shortcuts

Create or edit `~/.ssh/config` to set up shortcuts:

```
Host dgx01
    HostName 192.168.1.100
    User cynthia
    IdentityFile ~/.ssh/id_ed25519

Host dgx02
    HostName 192.168.1.101
    User cynthia
    Port 2222

Host bmc-dgx01
    HostName 192.168.1.200
    User root
```

Now you can just type:
```bash
$ ssh dgx01        # Instead of ssh -i ~/.ssh/id_ed25519 cynthia@192.168.1.100
$ ssh dgx02        # Connects on port 2222
$ ssh bmc-dgx01    # Connects to BMC as root
```

## SSH Permissions — This Will Save You Headaches

SSH is **very picky** about file permissions. If they're wrong, it silently refuses to work:

```bash
# Your .ssh directory must be 700
$ chmod 700 ~/.ssh

# Your private key must be 600 (owner read/write only)
$ chmod 600 ~/.ssh/id_ed25519

# Your public key can be 644
$ chmod 644 ~/.ssh/id_ed25519.pub

# authorized_keys must be 600
$ chmod 600 ~/.ssh/authorized_keys
```

If SSH authentication isn't working, check permissions first. This is the #1 cause of "why won't my key work."

## Copying Files with SCP

```bash
# Copy a file TO a remote server
$ scp report.txt cynthia@192.168.1.100:/home/cynthia/

# Copy a file FROM a remote server
$ scp cynthia@192.168.1.100:/var/log/nvidia/gpu-monitor.log ./

# Copy an entire directory (-r = recursive)
$ scp -r ./logs/ cynthia@192.168.1.100:/home/cynthia/logs/
```

## Useful SSH Tricks

### Running a Single Command Remotely
```bash
# Don't open an interactive session — just run one command
$ ssh dgx01 "nvidia-smi"
$ ssh dgx01 "tail -20 /var/log/syslog"
$ ssh dgx01 "df -h"
```

Great for quick checks without fully logging in.

### Port Forwarding
```bash
# Forward local port 8080 to remote port 80
$ ssh -L 8080:localhost:80 dgx01
# Now open http://localhost:8080 in your browser

# Forward local port 8888 to a Jupyter notebook on the remote server
$ ssh -L 8888:localhost:8888 dgx01
```

### Keep Connections Alive
Add to `~/.ssh/config`:
```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

This sends a keepalive packet every 60 seconds so your connection doesn't timeout.

## Troubleshooting SSH

```bash
# Verbose mode — shows exactly what's happening during connection
$ ssh -v cynthia@192.168.1.100

# Even more verbose
$ ssh -vvv cynthia@192.168.1.100
```

Verbose output tells you exactly where authentication is failing — wrong key, wrong permissions, server refusing connection, etc.

---

## Exercises

- [ ] Generate an SSH key pair with `ssh-keygen`
- [ ] Check the permissions on your `~/.ssh/` directory and key files
- [ ] Create a basic `~/.ssh/config` file with at least one host entry
- [ ] SSH into a server (even a local VM counts) and run `hostname`
- [ ] Use `scp` to copy a file to or from a remote server
- [ ] Try running a single command remotely: `ssh yourserver "uptime"`
- [ ] Run `ssh -v` against a server and read through the verbose output

## Review Questions

1. What's the difference between a public key and a private key? Which one do you share?
2. Why does SSH care so much about file permissions?
3. How would you set up a shortcut so `ssh dgx01` connects you to `admin@192.168.1.100`?
4. What command lets you copy a file from a remote server to your local machine?
5. You're getting "Permission denied (publickey)" when trying to SSH. What do you check first?

---

*Previous: [[M01L08 - Environment Variables & PATH]] | Next: [[M01L10 - Lab - Navigating a DGX via SSH]]*
