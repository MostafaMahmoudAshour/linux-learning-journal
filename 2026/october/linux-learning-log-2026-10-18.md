# Linux Learning Log — 2026-10-18

## 1. Linux architecture

A Linux distribution is a complete operating system built around the Linux kernel. Distributions share the Linux kernel family, but they may use different kernel versions, patches, configurations, and drivers.

```text
Applications
  Firefox · VS Code · Python · Media players
                |
System libraries and user-space tools
  glibc · Bash · core utilities · systemd · desktop environment
                |
          System call interface
                |
Linux kernel
  Process scheduling · Memory management · Filesystems
  Device drivers · Networking · Security · IPC
                |
Hardware
  CPU · RAM · SSD/HDD · GPU · Network card · Peripherals
```

The distribution is not just one layer: it assembles and configures the kernel, user-space software, package management, defaults, and often a desktop environment.

### What the kernel does

- **Process management and CPU scheduling:** manages processes and threads and schedules CPU time.
- **Memory management:** manages virtual memory, physical RAM, page tables, and memory pressure.
- **Filesystems and storage:** provides filesystem interfaces and coordinates storage operations.
- **Device drivers:** enables communication with supported hardware.
- **Networking:** implements core networking facilities such as TCP/IP and sockets.
- **Security and isolation:** enforces permissions and provides mechanisms such as capabilities and namespaces.
- **Inter-process communication (IPC):** provides mechanisms such as pipes, signals, and shared memory.
- **System calls:** let user-space programs request services from the kernel.

### Common kernel source directories

| Directory | Main area |
|---|---|
| `kernel/` | Core kernel mechanisms and scheduling |
| `mm/` | Memory management |
| `fs/` | Filesystems |
| `drivers/` | Device drivers |
| `net/` | Networking |
| `arch/` | Architecture-specific code |
| `block/` | Block storage I/O |
| `security/` | Security mechanisms |

The kernel does **not** include ordinary user-space programs such as Bash, `ls`, `cat`, GNOME, or package managers.

## 2. Filesystems: VFS, ext4, XFS, Btrfs

A filesystem organizes data and metadata on storage devices.

- **VFS (Virtual File System):** a Linux kernel subsystem that provides a common interface for accessing different filesystem types. Applications can use familiar file operations without needing to know the filesystem's internal implementation.
- **ext4:** a mature, widely used general-purpose Linux filesystem. It supports journaling and is a common default choice.
- **XFS:** a journaling filesystem well suited to large filesystems and storage-intensive workloads. It supports online expansion but cannot be shrunk.
- **Btrfs (B-tree Filesystem):** supports copy-on-write, snapshots, subvolumes, checksums, and compression.

| Filesystem | Notable strengths |
|---|---|
| ext4 | Mature, dependable general-purpose choice |
| XFS | Scales well for large storage workloads |
| Btrfs | Snapshots and advanced data-management features |

VFS is the common interface, not a filesystem format you select when formatting a drive.

## 3. Desktop environments

A desktop environment is user-space software that provides the graphical desktop: panels, menus, window management, settings, and desktop applications.

- **GNOME:** modern, streamlined interface; commonly used by Ubuntu and Fedora Workstation.
- **KDE Plasma:** highly customizable; used by options such as Kubuntu.
- **Xfce:** relatively lightweight, traditional desktop layout.
- **Cinnamon:** traditional menu-and-panel experience, commonly associated with Linux Mint.

| Desktop | General style | Main appeal |
|---|---|---|
| GNOME | Modern and streamlined | Consistent workflow |
| KDE Plasma | Highly customizable | Flexibility |
| Xfce | Traditional and lightweight | Efficiency |
| Cinnamon | Traditional and familiar | Familiar desktop layout |

A computer can run the same kernel with different desktop environments. You can also install multiple desktop environments on one installation, though this can add duplicate apps and settings.

## 4. Common Linux distribution families

A distribution packages the kernel with user-space tools, libraries, package management, configuration, and other software.

### Debian family

- **Debian:** large software repository; known for its stable release.
- **Ubuntu:** Debian-based; popular for desktops, development, and servers.
- **Linux Mint:** its main edition is Ubuntu-based; known for its Cinnamon desktop.
- **Pop!_OS:** Ubuntu-based; focuses on desktop productivity and developer workflows.
- **Kali Linux:** Debian-based; specialized for penetration testing and security auditing.

Common package format/tooling: `.deb` packages and APT.

### Red Hat family

- **Fedora:** adopts many newer Linux technologies early.
- **Red Hat Enterprise Linux (RHEL):** enterprise distribution with commercial support.
- **Rocky Linux:** designed for compatibility with RHEL.
- **AlmaLinux:** another RHEL-compatible enterprise distribution.

Common package format/tooling: RPM packages and DNF. Fedora is an important upstream project for RHEL; Rocky Linux and AlmaLinux are RHEL-compatible rebuilds, not direct Fedora derivatives.

### Arch family

- **Arch Linux:** rolling release, customizable, uses `pacman`.
- **Manjaro:** aims to make an Arch-based desktop more approachable.
- **EndeavourOS:** offers an easier installation path while staying close to Arch.
- **Garuda Linux:** Arch-based with a customized desktop experience.

### Other important distributions

- **openSUSE:** offers Leap and Tumbleweed; uses tools such as Zypper and YaST.
- **Gentoo:** highly configurable, source-oriented package management.
- **Slackware:** traditional design and hands-on administration.
- **NixOS:** declarative system configuration and reproducible management.
- **Alpine Linux:** lightweight, security-conscious, often used in containers.

Simplified family tree:

```text
Debian
└── Ubuntu
    ├── Linux Mint (main edition)
    ├── Pop!_OS
    └── elementary OS

Fedora ── upstream relationship ──> RHEL
                                  ├── Rocky Linux
                                  └── AlmaLinux

Arch Linux
├── Manjaro
├── EndeavourOS
└── Garuda Linux

Independent projects/families include openSUSE, Gentoo, Slackware,
NixOS, and Alpine Linux.
```

This is simplified: distribution histories and relationships can be more complex.



## 5. Terminal prompt: `username@hostname`

Example:

```text
mostafa@ubuntu-pc:~$
```

| Part | Meaning |
|---|---|
| `mostafa` | Current username |
| `@` | Separates username and hostname |
| `ubuntu-pc` | Hostname — the computer's assigned name |
| `~` | Current directory is the user's home directory |
| `$` | Normal user's shell prompt (commonly) |

A hostname identifies a computer, particularly on a network. It is not the same thing as an IP address or an internet domain name.

### Commands to know

```bash
hostname                    # Print the hostname
hostnamectl                 # Show/manage hostname and system information
whoami                      # Print the current username
pwd                         # Print the current working directory
uname -r                    # Show the running kernel version
uname -a                    # Show general kernel/system information
cat /etc/os-release         # Show distribution information
lsmod                       # List loaded kernel modules
lscpu                       # Show CPU information
free -h                     # Show memory usage in human-readable units
systemctl --version         # Show systemd version, if installed
apt --version               # Show APT version, if installed
ps -p 1 -o comm=            # Show the command name of process ID 1
```

To change the hostname on Ubuntu:

```bash
sudo hostnamectl set-hostname ml-server
```

Choose a meaningful hostname; this changes the system's assigned hostname.

## 6. What happens when you run `cat notes.txt`?

1. Bash reads and parses the command.
2. Bash starts the `cat` program.
3. `cat` requests file access through library functions and system calls.
4. The kernel checks permissions and coordinates filesystem/storage operations.
5. `cat` writes the file contents to standard output, and the terminal displays them.

Bash and `cat` are user-space programs; the kernel provides the lower-level services they use.

## Key takeaways

- The **kernel** manages hardware and core system resources.
- A **distribution** assembles a usable operating system around the kernel.
- **VFS** provides a common filesystem interface; ext4, XFS, and Btrfs are filesystem implementations.
- **GNOME, KDE Plasma, Xfce, and Cinnamon** are desktop environments, not kernels.
- Distribution families often differ in package managers, defaults, release models, and target users.
- In `username@hostname`, the part after `@` is the computer's hostname.

## Learning checklist

- [ ] Explain the difference between kernel and distribution.
- [ ] Name at least four kernel responsibilities.
- [ ] Explain VFS versus a filesystem such as ext4.
- [ ] Distinguish filesystem types from desktop environments.
- [ ] Identify Debian-, Fedora/RHEL-, and Arch-based distributions.
- [ ] Use `hostname`, `whoami`, `pwd`, `uname -r`, and `cat /etc/os-release`.
