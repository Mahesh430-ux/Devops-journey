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
