# Devops-journey :- 
# Linux Basics: File System, File Management & File Processing

My Day 1 notes while learning Linux. In Linux, everything is a file.

![Linux File System Cheat Sheet](<img width="1505" height="1600" alt="linux-cheatsheet-1" src="https://github.com/user-attachments/assets/66673f4f-6d0e-4f24-a337-66041950c671" />
)

## 1. Basic File System

| Directory | Purpose |
|-----------|---------|
| `/` | Root: top of the whole tree |
| `/bin` | Essential commands (`ls`, `cp`, `cat`) |
| `/etc` | System config files |
| `/home` | User home directories |
| `/root` | Home of the root user |
| `/var` | Logs and changing data |
| `/tmp` | Temporary files |
| `/usr` | Apps and libraries |
| `/dev` | Device files (disk, USB) |
| `/proc` | Process and kernel info |
| `/opt` | Optional software |
| `/mnt` | Mounted drives |

**Paths:** `/home/user/a.txt` (absolute), `./docs/a.txt` (relative), `~` (home), `.` (current dir), `..` (parent dir)

## 2. File Management

| Command | What it does |
|---------|--------------|
| `pwd` | Show current directory |
| `ls -la` | List all files with details |
| `cd <dir>` | Change directory |
| `mkdir -p` | Create directories |
| `touch file` | Create an empty file |
| `cp -r` | Copy files / folders |
| `mv` | Move or rename |
| `rm -r` | Delete (no undo!) |
| `rmdir` | Remove an empty folder |
| `find . -name` | Search for files |
| `ln -s` | Create a symlink |
| `du -sh` | Show folder size |

## 3. File Processing

| Command | What it does |
|---------|--------------|
| `cat` | View file contents |
| `less` | Scroll through big files |
| `head -n 5` | First 5 lines |
| `tail -f` | Follow a log live |
| `grep "text"` | Search inside files |
| `wc -l` | Count lines / words |
| `sort` | Sort lines |
| `uniq` | Remove duplicate lines |
| `cut -d, -f1` | Extract columns |
| `sed 's/a/b/'` | Find and replace text |

## 4. File Permissions

-rwxr-xr--
 │││ │││ │││
 │││ │││ └┴┴─ Others: r-- = 4
 │││ └┴┴───── Group:  r-x = 5
 └┴┴───────── Owner:  rwx = 7
```

`r = 4`, `w = 2`, `x = 1`


chmod 754 script.sh


## 5. Redirection & Pipes

ls > files.txt            # redirect output (overwrite)
echo "hi" >> log.txt      # append to a file
sort < names.txt          # take input from a file
ls | wc -l                # pipe output to the next command

# Combine them
cat access.log | grep 404 | sort | uniq -c
```

---

Learning Linux, one command at a time.

`#Linux` `#LearningInPublic` `#OpenSource` `#DevOps` `#SysAdmin`





# Day 02 – ACL Permissions, Linux Directories, grep & Troubleshooting

## What I learned

### 1. ACL permissions
- View ACL: `getfacl file`
- Give a user access: `setfacl -m u:username:rw file`
- Remove a user's entry: `setfacl -x u:username file`
- ACLs allow access control beyond owner/group/other.

### 2. Important directories
| Directory | Purpose |
|-----------|---------|
| /bin | Essential user commands |
| /tmp | Temporary files |
| /root | Home directory of the root user |
| /dev | Device files |

### 3. grep
- `grep "error" file.log` – search for a word
- `grep -i "error" file.log` – ignore case
- `grep -r "text" /path` – search recursively

### 4. Troubleshooting practice
1. Read the error message carefully.
2. Check permissions with `ls -l` and `getfacl`.
3. Search logs with `grep`.
4. Fix with `chmod` or `setfacl`.
5. Test again to confirm it works.

## Key takeaway
Understanding the file system and permissions makes troubleshooting much faster.


# Linux Troubleshooting & File Permissions

> Hands-on notes from my Linux learning journey. Today I explored real-world troubleshooting problems and how file permissions work, with the commands and fixes I used.

## Table of Contents

- [What I learned today](#what-i-learned-today)
- [Linux troubleshooting problems I explored](#linux-troubleshooting-problems-i-explored)
- [File permissions deep dive](#file-permissions-deep-dive)
- [Hands-on practice labs](#hands-on-practice-labs)
- [Common mistakes](#common-mistakes)
- [Quick cheat sheet](#quick-cheat-sheet)

---

## What I learned today

- A good troubleshooting flow is: **observe, check logs, test a hypothesis, fix, verify**.
- Most "Permission denied" errors come from three things: wrong mode, wrong owner, or a missing `x` on a parent directory.
- Logs (`journalctl`, `dmesg`, `/var/log`) almost always tell you what went wrong.
- `chmod 777` is never the right fix.

---

## Linux troubleshooting problems I explored

### 1. Disk full ("No space left on device")

**Problem:** The app crashes and `df` shows 100% used, even after deleting big files.

**Diagnose**
```bash
df -h                              # space per filesystem
df -i                              # inodes exhausted?
du -xh / 2>/dev/null | sort -rh | head -15   # biggest directories
lsof +L1                           # deleted files still held open by a process
```

**Fix:** Clean logs and caches (`apt clean`, `journalctl --vacuum-size=200M`), restart the process holding deleted files, and set a disk alert at 80%.

---

### 2. High CPU load

**Problem:** Load average is 12 on a 4-core server and the app crawls.

**Diagnose**
```bash
uptime                             # load average
ps aux --sort=-%cpu | head -10     # top CPU processes
top -o %CPU                        # live view
```

**Fix:** Find the culprit, fix or restart it, lower priority with `renice +10 -p <PID>`, and add CPU limits (cgroups).

---

### 3. SSH: "Connection refused"

**Problem:** `ping` works, but `ssh user@host` says "Connection refused".

**Diagnose**
```bash
systemctl status sshd              # is sshd running?
ss -tlnp | grep ssh                # listening on port 22?
sshd -t                            # validate config syntax
sudo ufw status || firewall-cmd --list-all   # firewall blocking?
```

**Fix:** `systemctl enable --now sshd`, fix config errors, and open port 22 in the OS firewall and the cloud security group.

---

### 4. Service fails to start

**Problem:** `systemctl start nginx` returns "Job failed".

**Diagnose**
```bash
systemctl status nginx
journalctl -xeu nginx --no-pager | tail -30
nginx -t                           # test config before starting
ss -tlnp | grep ':80'              # port already in use?
```

**Fix:** Fix the config error or port conflict shown in the logs, then `systemctl daemon-reload && systemctl restart nginx`.

---

### 5. OOM killer killed my process

**Problem:** A database dies randomly at peak hours with no crash log.

**Diagnose**
```bash
dmesg -T | grep -i 'killed process'
free -h
ps aux --sort=-%mem | head -10
```

**Fix:** Fix the memory leak, add swap as a safety net, set memory limits, and right-size the instance.

---

### 6. DNS not resolving

**Problem:** `ping 8.8.8.8` works, but `ping google.com` fails.

**Diagnose**
```bash
dig google.com @8.8.8.8
resolvectl status
cat /etc/resolv.conf
```

**Fix:** Set a working nameserver persistently (netplan or NetworkManager) and run `sudo systemctl restart systemd-resolved`.

---

### 7. Cron job not running

**Problem:** The script works manually, but cron never runs it.

**Diagnose**
```bash
crontab -l
systemctl status cron
grep CRON /var/log/syslog | tail
env -i /bin/sh -c '/opt/backup.sh'   # test with a minimal environment
```

**Fix:** Use absolute paths, set `PATH` inside the script, `chmod +x` it, and log output: `*/5 * * * * /opt/backup.sh >> /tmp/backup.log 2>&1`.

---

## File permissions deep dive

### Reading `ls -l`

```text
-rwxr-xr--  1  alice  devs  4096  Oct 18 10:00  deploy.sh
│└┬┘└┬┘└┬┘     │      │
│ │  │  │      │      └── group
│ │  │  │      └── owner
│ │  │  └── others: r--
│ │  └── group:  r-x
│ └── owner:  rwx
└── file type (- file, d directory, l symlink)
```

| Symbol | Meaning on a file | Meaning on a directory |
|--------|-------------------|------------------------|
| `r` (4) | read contents | list entries |
| `w` (2) | modify contents | create/delete entries |
| `x` (1) | execute | enter (`cd`) and access items |

### chmod: numeric vs symbolic

```bash
chmod 755 script.sh        # numeric: rwxr-xr-x
chmod u+x script.sh        # symbolic: add execute for owner
chmod go-w file.txt        # remove write for group and others
chmod -R 750 /opt/app      # recursive
```

| Octal | Symbolic | Typical use |
|-------|----------|-------------|
| 755 | rwxr-xr-x | scripts, public directories |
| 644 | rw-r--r-- | normal files, web content |
| 600 | rw------- | private keys, secrets |
| 700 | rwx------ | private directories (e.g. `~/.ssh`) |

### chown and chgrp

```bash
sudo chown alice file.txt              # change owner
sudo chown alice:devs file.txt         # change owner and group
sudo chown -R www-data:www-data /var/www/html
sudo chgrp devs shared/                # change group only
```

### umask (default permissions)

```bash
umask            # show current mask, e.g. 0022
umask 027        # new files: 640, new directories: 750
```

New files start from 666 and directories from 777, then the umask bits are removed. With `022`: files become 644 and directories 755.

### Special bits

| Bit | Octal | Effect | Example |
|-----|-------|--------|---------|
| setuid | 4000 | file runs as its owner | `/usr/bin/passwd` |
| setgid | 2000 | file runs as its group; on a directory, new files inherit the group | shared team folders |
| sticky | 1000 | only the owner can delete files in the directory | `/tmp` |

```bash
chmod u+s program          # setuid
chmod 2775 /srv/team       # setgid directory
chmod +t /srv/uploads      # sticky bit
find / -perm -4000 -type f 2>/dev/null   # audit setuid files
```

### ACLs (finer-grained control)

```bash
getfacl report.txt
setfacl -m u:bob:rw report.txt      # give bob read/write
setfacl -m g:auditors:r report.txt  # give a group read access
setfacl -x u:bob report.txt         # remove bob's entry
setfacl -b report.txt               # remove all ACLs
```

A `+` at the end of the mode in `ls -l` means an ACL is set.

### SELinux contexts

```bash
getenforce                          # Enforcing / Permissive / Disabled
ls -Z /var/www/html                 # show contexts
sudo restorecon -Rv /var/www/html   # restore default contexts
ausearch -m avc -ts recent          # recent SELinux denials
```

If permissions look right but access is still denied on RHEL/CentOS/Fedora, SELinux is often the reason.

### Debugging "Permission denied" step by step

```bash
ls -l deploy.sh                     # 1. check mode and owner
namei -l /opt/app/deploy.sh         # 2. check EVERY directory in the path
id                                  # 3. who am I, which groups?
ls -Z deploy.sh && getenforce       # 4. SELinux context
mount | grep noexec                 # 5. is the filesystem mounted noexec?
```

---

## Hands-on practice labs

### Lab 1: Make a script executable
```bash
echo -e '#!/bin/bash\necho "Hello"' > hello.sh
./hello.sh            # Permission denied
chmod +x hello.sh
./hello.sh            # Hello
```

### Lab 2: Shared team directory with setgid
```bash
sudo groupadd devs
sudo mkdir /srv/team
sudo chgrp devs /srv/team
sudo chmod 2775 /srv/team
touch /srv/team/a.txt && ls -l /srv/team   # file inherits group "devs"
```

### Lab 3: Fix a web root owned by root
```bash
ls -ld /var/www/html                       # owner root, nginx cannot write
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```

### Lab 4: Missing `x` on a parent directory
```bash
mkdir -p lab4/inner && echo hi > lab4/inner/file.txt
chmod 600 lab4                              # remove x from the parent
cat lab4/inner/file.txt                     # Permission denied
namei -l lab4/inner/file.txt                # shows where it breaks
chmod 755 lab4                              # fix
```

### Lab 5: Give one extra user access with an ACL
```bash
touch secret.txt && chmod 600 secret.txt
setfacl -m u:bob:r secret.txt
getfacl secret.txt                          # bob has r--
```

---

## Common mistakes

- Running `chmod 777` to "make it work". It hides the real problem and is a security risk.
- Forgetting that directories need `x` to be entered, on every level of the path.
- Using `chown -R` or `chmod -R` on `/` or on a broad path by accident.
- Making private keys group- or world-readable (SSH refuses them; keep them at `600`).
- Ignoring SELinux and ACLs when the normal permissions look fine.
- Editing `/etc/fstab` without running `mount -a` before rebooting.

---

## Quick cheat sheet

| Task | Command |
|------|---------|
| Show permissions | `ls -l`, `stat file` |
| Change mode | `chmod 644 file` / `chmod u+x file` |
| Change owner | `chown user:group file` |
| Show path permissions | `namei -l /full/path` |
| Show my user and groups | `id` |
| Default permission mask | `umask` |
| View ACLs | `getfacl file` |
| Set ACL | `setfacl -m u:user:rw file` |
| SELinux context | `ls -Z file` / `restorecon -Rv path` |
| Find setuid files | `find / -perm -4000 -type f 2>/dev/null` |
| Disk usage | `df -h`, `du -xh / | sort -rh | head` |
| Service logs | `journalctl -xeu <service>` |
| Kernel messages | `dmesg -T` |

---

*Learning in public. If this helped, give the repo a star and follow my journey!*

