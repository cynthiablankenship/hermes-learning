# M01L05 - Permissions & Ownership

## The Big Idea

Linux is a multi-user system. Permissions control who can read, write, or execute files. When you see "Permission denied," it's permissions stopping you.

## Reading Permission Strings

Every file and directory has a permission string. You see it with `ls -l`:

```
-rwxr-xr-- 1 cynthia users 4096 Apr 9 10:00 script.sh
```

Let's break down `-rwxr-xr--`:

```
- rwx r-x r--
│  │   │   │
│  │   │   └── Other (everyone else): read only
│  │   └── Group: read + execute
│  └── Owner (cynthia): read + write + execute
└── File type (- = file, d = directory)
```

Each position is either a letter or `-`:

| Position | Letter | Means |
|----------|--------|-------|
| 1 | `r` | Read — can view the contents |
| 2 | `w` | Write — can modify the contents |
| 3 | `x` | Execute — can run as a program (or enter if directory) |

So `rwxr-xr--` means:
- **Owner** can read, write, execute
- **Group** can read, execute
- **Others** can only read

## Common Permission Patterns

| String | Meaning | When You'll See It |
|--------|---------|-------------------|
| `-rw-r--r--` | Owner can edit, everyone can read | Most files |
| `-rwxr-xr-x` | Everyone can run | Scripts, programs |
| `drwxr-xr-x` | Directory everyone can enter | Most directories |
| `-rw-------` | Only owner can read/write | SSH private keys, secrets |
| `-r--------` | Read-only, only owner | Sensitive configs |

## Changing Permissions with `chmod`

### Symbolic Mode
```bash
# Add execute permission for the owner
$ chmod u+x script.sh

# Remove write permission for others
$ chmod o-w file.txt

# Give group read+write
$ chmod g+rw file.txt

# Set exact permissions for everyone
$ chmod u=rwx,g=rx,o=r script.sh
```

Letters: `u` = user/owner, `g` = group, `o` = others, `a` = all
Operators: `+` = add, `-` = remove, `=` = set exactly

### Numeric Mode (Octal)
Each permission combination maps to a number:

| Permission | Binary | Octal |
|-----------|--------|-------|
| `---` | 000 | 0 |
| `--x` | 001 | 1 |
| `-w-` | 010 | 2 |
| `-wx` | 011 | 3 |
| `r--` | 100 | 4 |
| `r-x` | 101 | 5 |
| `rw-` | 110 | 6 |
| `rwx` | 111 | 7 |

```bash
# rwxr-xr-x = 755
$ chmod 755 script.sh

# rw-r--r-- = 644
$ chmod 644 file.txt

# rw------- = 600
$ chmod 600 secret.key

# r-------- = 400
$ chmod 400 ~/.ssh/id_rsa
```

**You'll see these numbers a lot.** The most common ones to remember:
- **755** — executable programs/scripts
- **644** — regular files
- **600** — private files (SSH keys, configs with passwords)

## Changing Ownership with `chown`

```bash
# Change owner
$ sudo chown cynthia file.txt

# Change owner and group
$ sudo chown cynthia:users file.txt

# Recursively change ownership of a directory
$ sudo chown -R cynthia:users /opt/project/
```

Note: You usually need `sudo` (admin privileges) to change ownership.

## Permission Denied — What's Happening

When you get:
```
bash: ./script.sh: Permission denied
```

It usually means one of:
1. The file isn't executable → `chmod +x script.sh`
2. You don't own the file → `sudo chown cynthia script.sh` or use `sudo` to run it
3. The directory doesn't allow you to enter it → check directory permissions

## The `sudo` Command

`sudo` = "superuser do." It runs a command as root (admin). You'll be prompted for your password.

```bash
$ sudo cat /var/log/secure-log.txt       # Read a file only root can read
$ sudo systemctl restart docker           # Restart a service (needs admin)
$ sudo apt install nvidia-driver-535      # Install software
```

**Be careful with sudo.** It bypasses all permission checks. `sudo rm -rf /` would delete everything.

---

## Exercises

- [ ] Create a file and check its permissions with `ls -l`
- [ ] Use `chmod` to make the file executable (both symbolic and numeric)
- [ ] Remove all permissions from a file, then try to read it (you'll get permission denied)
- [ ] Fix the permissions so you can read it again
- [ ] Check the permissions on `/etc/shadow` — why can't you read it normally?
- [ ] Try reading it with `sudo cat /etc/shadow`

## Review Questions

1. What does `chmod 755 script.sh` set the permissions to?
2. What's wrong if you try `./script.sh` and get "Permission denied"?
3. What permission (in octal) should your SSH private key have and why?
4. What's the difference between `chmod u+x` and `chmod +x`?
5. Why does `sudo` let you bypass permissions?

---

*Previous: [[M01L04 - Reading & Searching Files]] | Next: [[M01L06 - Processes & Monitoring]]*
