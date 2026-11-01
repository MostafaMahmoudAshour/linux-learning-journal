## Table of Contents

1. [Users in Linux](#1-users-in-linux)
2. [Groups in Linux](#2-groups-in-linux)
3. [File Management](#3-file-management)
4. [Inodes & Links](#4-inodes--links)
5. [Filesystem Hierarchy](#5-filesystem-hierarchy)
6. [File Types](#6-file-types)
7. [Searching for Files & Directories](#7-searching-for-files--directories)
8. [Archiving & Compression](#8-archiving--compression)

---

## 1. Users in Linux

A **user** is an account with a set of permissions. Linux has three types of users:

| Type | Description | UID Range |
|---|---|---|
| **Superuser (root)** | Full permissions — read/write any file, install/uninstall software, manage the system | `0` |
| **System User** | Used by software/services, not humans | `1` – `999` |
| **Normal User** | The everyday human login account | `1000+` |

Each user has:
- **UID** (User ID)
- **Shell** (default login shell)
- **Home Directory**
- Other settings

### Viewing users

```bash
ls /home                  # list local user home directories
cat /etc/passwd           # all users with details:
# userName:password:userId:primaryGroupID:comment:HomeDirectory:DefaultShell

cat /etc/passwd | grep mostafa
# mostafa:x:1000:1000:,,,:/home/mostafa:/bin/bash

cat /etc/passwd | grep root
# root:x:0:0:root:/root:/bin/bash     → root has UID=0

cat /etc/shadow           # passwords + more details — requires superuser access
```

### Switching users

```bash
su - root     # or: su / su - / su -l   (su = Substitute User)
```

> You must set a root password before you can `su` into root (see below).

### The `sudoers` file

If your username is listed in the **sudoers** file, you can use the `sudo` command. It asks for **your own login password**, then executes the command with **root privileges**.

> `sudo` = **S**witch **U**ser **DO**

### Reset / set the root password

```bash
sudo passwd root       # or simply: sudo passwd
```

1. Enter your **current user password** (to obtain `sudo` privileges).
2. Enter and confirm the **new root password**.
3. You should see: `password updated successfully`.

### Creating a user

```bash
useradd userName -c "this is a comment"
```

- Run as **root**.
- Verify with: `cat /etc/passwd | grep userName`
- `-m` → create a home directory for the user (also appears in `ls /home`):
  ```bash
  useradd Mostafa -c "this is me" -m
  ```
- `-p` → set a password:
  ```bash
  useradd Test -p P@ssw0rd
  ```

Check users and whether they have passwords:

```bash
cat /etc/shadow | tail -n 5
```

### Other user commands

```bash
passwd userName        # reset a user's password (then enter the new password)
userdel userName       # delete a user
usermod                # modify user settings
```

---

## 2. Groups in Linux

Instead of assigning permissions to each user individually, we put users into **groups** and assign permissions to the group. Every user in that group inherits its permissions.

```bash
cat /etc/group   # database file containing group info:
                 # groupName : groupPassword : groupID : members

groups           # lists all groups the current user belongs to
```

### Key concepts

- When you create a user with `useradd`, Linux automatically creates a group with the **same name** as the user. This is the **Primary Group** (with a Primary Group ID).
- You can specify a different primary group at creation time with `-g`.

### Managing groups

```bash
# Create a group
sudo groupadd group1
cat /etc/group | grep group1     # verify: group1:x:1005:

# Add a user to a group (append to secondary groups)
sudo usermod -aG group1 sami

# Remove a user from a group
sudo usermod -rG group1 sami

# Delete a group
sudo groupdel group1
```

> **Note:** `-a` (append) is critical with `-G`. Without it, `usermod -G` **overwrites** the user's existing secondary groups.

---

## 3. File Management

### Paths

- **Absolute path** — starts from `/` (e.g., `/var/log`)
- **Relative path** — relative to current directory (e.g., `../log`)

### Listing & inspecting files

```bash
ls -l          # long listing
ls -lh         # human-readable sizes
ls -lhd        # show directory entry itself, not its contents
ls -R          # recursive listing (directory hierarchy)
sudo tree      # like ls -R but more organized output

stat fileName  # detailed statistics/metadata about a file
```

### Useful commands

```bash
which command    # locate the path of a specific command
uname -r         # show installed kernel version
file fileName    # determine the type of a file

ls -l | more     # paginate long output (page by page)
less fileName    # paginate, but scroll line by line with arrow keys
```

### Creating nested directories

```bash
mkdir -p dir1/dir2/dir3   # -p creates parent directories as needed
```

---

## 4. Inodes & Links

Every file in Linux consists of three things:

| Component | Description |
|---|---|
| **Filename** | The name/tag of the file |
| **Data block** | The actual content stored on disk |
| **Inode** | Metadata about the file (permissions, owner, size, timestamps, etc.) |

- The inode number (e.g., `5334`) is the **index** of that inode.
- View metadata with `stat fileName`.

### Links

Links in Linux are like **shortcuts** in Windows. There are two types:

#### Hard Links

Share the **same data block and same inode** — just a different name. Like one box with two tags.

```bash
ln original hardlink     # ln existingFile newLinkFile

ls -il original hardlink
# 13538 -rw-r--r-- 2 mostafa mostafa 0 Oct 26 15:39 hardlink
# 13538 -rw-r--r-- 2 mostafa mostafa 0 Oct 26 15:39 original
# Same inode (13538), same size, same everything except the name.

stat original
stat hardlink   # identical except for the filename
```

#### Soft Links (Symbolic Links)

Like a Windows shortcut — accessing it redirects you to the original file.

```bash
ln -s original symlink    # -s for symbolic

ls -il original symlink
# 13538 -rw-r--r-- 2 mostafa mostafa 0 Oct 26 15:39 original
#  7778 lrwxrwxrwx 1 mostafa mostafa 8 Oct 27 15:02 symlink -> original
# Different inode, permissions, name — but points to the same content.
```

#### Behavior comparison

| Action | Hard Link | Soft Link |
|---|---|---|
| Delete original | Still works with full content | Becomes broken (dangling), but the link file still exists |
| Inode | Same as original | Different |
| Cross filesystems | ❌ Not allowed | ✅ Allowed |

---

## 5. Filesystem Hierarchy

Linux organizes everything into a single inverted tree starting at `/` (the **root directory** — not to be confused with `/root`, the root user's home).

> Unlike Windows, there are no drive letters. Everything is mounted somewhere under `/`.

### Device naming

Devices live in `/dev`. The name starts with letters indicating the device type:

- SATA disk → `sd`
- First SATA disk → `sda`
- First partition on it → `sda1`

So the first SATA disk is `/dev/sda`, and its first partition is `/dev/sda1`.

### Mounting

In Windows, `C:\users` is always on the same disk/partition as `C:\`. **In Linux, this is not the case.** Even though everything appears under `/`, different directories can live on different partitions or even different disks.

For example:
- `/home` can be on one partition of a disk
- `/home/mostafa` can be on a different partition of the same disk, or on a completely different disk

This is called **Mounting**.

### Key directories

| Directory | Purpose |
|---|---|
| `/bin` | Essential user binaries (`ls`, `cp`, `bash`) — often symlink to `/usr/bin` |
| `/sbin` | System binaries for administration (`fdisk`, `reboot`) |
| `/boot` | Kernel (`vmlinuz`), initrd, GRUB config |
| `/etc` | System-wide configuration files (`/etc/passwd`, `/etc/fstab`, `/etc/hosts`) |
| `/home` | User home directories (`/home/ahmed`, `/home/maria`) |
| `/root` | Root user's home (separate from `/home`) |
| `/tmp` | Temporary files — cleared on reboot, world-writable |
| `/var` | Variable data: `/var/log`, `/var/www`, `/var/lib`, `/var/cache` |
| `/usr` | User system resources: `/usr/bin`, `/usr/lib`, `/usr/share`, `/usr/local` |
| `/lib` | Shared libraries needed by `/bin` and `/sbin` |
| `/opt` | Optional/third-party software (e.g., `/opt/google/chrome`) |
| `/dev` | Device files (`/dev/sda`, `/dev/tty`, `/dev/null`) |
| `/proc` | **Virtual** filesystem — live process & kernel info |
| `/sys` | **Virtual** filesystem — kernel & hardware info |
| `/mnt` | Manual mount points |
| `/media` | Automatic mount points for removable media (USB, CD) |
| `/run` | Runtime data (PID files, sockets) — tmpfs, cleared on reboot |
| `/srv` | Service data (web, FTP) |

### Visual overview

```
/
├── bin/        → essential user commands
├── boot/       → kernel & bootloader
├── dev/        → device files
├── etc/        → configuration files
├── home/       → user home directories
│   ├── ahmed/
│   └── maria/
├── lib/        → shared libraries
├── media/      → removable media mounts
├── mnt/        → manual mount points
├── opt/        → optional/third-party software
├── proc/       → virtual: process info
├── root/       → root user's home
├── run/        → runtime data
├── sbin/       → system binaries
├── srv/        → service data
├── sys/        → virtual: system/hardware info
├── tmp/        → temporary files
├── usr/        → user programs & data
│   ├── bin/
│   ├── lib/
│   ├── local/
│   └── share/
└── var/        → variable data (logs, caches)
    ├── log/
    ├── cache/
    └── www/
```

### The FHS Standard

This layout follows the **Filesystem Hierarchy Standard (FHS)**, maintained by the Linux Foundation. Most distributions (Debian, Ubuntu, Fedora, RHEL, Arch) follow it closely with minor variations.

### Exploring `/proc`

```bash
cd /proc
cat cpuinfo     # CPU information
cat meminfo     # Memory information
```

### Practical lookup table

| Task | Where to look |
|---|---|
| Find a command | `/bin`, `/usr/bin`, `/usr/local/bin` |
| Edit a service config | `/etc/` |
| Check logs | `/var/log/` |
| Install custom software | `/opt/` or `/usr/local/` |
| Check disk/memory info | `/proc/` or `/sys/` |
| User's personal files | `/home/username/` |

### Common confusions

- **`/bin` vs `/usr/bin`**: On modern systems, `/bin` is often a symlink to `/usr/bin` (the "usr merge"). Historically, `/bin` held boot-critical files.
- **`/proc` and `/sys` are not real files** — they're kernel interfaces for live system info.
- **`/root` ≠ `/`** — `/root` is the root user's home directory; `/` is the top of the filesystem.

---

## 6. File Types

Look at the first character of a permission string:

```bash
dr-xr-xr-x  3 root  root  0 Oct 27 15:44 scsi
```

- `d` at the start → **directory**
- `-` at the start → **regular file**
- `l` → symbolic link

Determine type explicitly:

```bash
file fileName
```

---

## 7. Searching for Files & Directories

### Using `find`

```bash
find whereToStartSearch -type fileOrDirectory -name pattern

find . -type f -name "*.log"     # files with .log extension from current dir
find . -type f -name "d*.log"
find . -type f -name "d??.log"   # ? matches exactly one character

# Perform operations on found files:
cat $(find . -type f -name "d??.log")
```

### Using `xargs`

Takes the output of a command and passes it as arguments to another command.

```bash
touch file{1,2,3}                     # creates file1 file2 file3

ls file? | xargs -I {} mkdir {}.d     # {} iterates over each item
# -I {} → replace string placeholder
# Result: file1.d file2.d file3.d
```

### Using `locate`

Like `find`, but searches a prebuilt database — much faster and lightweight. However, **not available on all distros**.

---

## 8. Archiving & Compression

### `.zip` vs `.tar`

- **`.zip`** — bundles all files into one file **and compresses** them.
- **`.tar`** — bundles all files into one file; **compression is optional**.

### Creating & extracting tar archives

```bash
# Create archive
tar -cf file.tar fileToArchive    # -c create, -f archive filename
tar -cf dir.tar dir/

# Verbose (show details during creation)
tar -cvf dir.tar dir/

# List contents without extracting
tar -tf dir.tar

# Extract
tar -xf dir.tar
tar -xf dir.tar -C /home/mostafa/Test    # -C extract to another location
```

### Compression with tar

```bash
# gzip compression
tar -czvf dir.tar.gz dir/
mv dir.tar.gz dir.tgz          # rename is fine

# bzip2 compression
tar -cjvf dir.tar.bz dir/

# Extract compressed archives
tar -xjvf dir.tar.bz
tar -xjvf dir.tar.bz -C /home/mostafa/dir

# Another example
tar -cjvf /var/log.tar.bz /var/log/
```

### Standalone gzip / gunzip

```bash
gzip output.txt          # compresses in place → output.txt.gz (original replaced)
gzip -k output.txt       # -k keeps the original file
gzip -d output.txt.gz    # -d decompress
```

---

## Quick Reference Cheat Sheet

| Task | Command |
|---|---|
| List all users | `cat /etc/passwd` |
| See user's groups | `groups` |
| Switch to root | `su - root` |
| Run command as root | `sudo command` |
| Set root password | `sudo passwd root` |
| Create user | `useradd name -m -c "comment"` |
| Set user password | `passwd name` |
| Delete user | `userdel name` |
| Create group | `groupadd name` |
| Add user to group | `usermod -aG group user` |
| Remove user from group | `usermod -rG group user` |
| Create hard link | `ln original link` |
| Create soft link | `ln -s original link` |
| Show file metadata | `stat file` |
| Show inode | `ls -i file` |
| Locate a command | `which cmd` |
| Show kernel version | `uname -r` |
| Determine file type | `file name` |
| Find files | `find . -type f -name "*.log"` |
| Create nested dirs | `mkdir -p a/b/c` |
| Create tar archive | `tar -cvf archive.tar dir/` |
| Extract tar archive | `tar -xvf archive.tar` |
| Compress with gzip | `gzip -k file` |

---

*Notes compiled while learning Linux fundamentals.*