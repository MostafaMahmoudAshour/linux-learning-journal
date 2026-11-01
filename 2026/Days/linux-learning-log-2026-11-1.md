# Linux Service Management & Package Management Notes

> A summary of Linux service/daemon management and software package management across distributions.

## Table of Contents
- [Service / Daemon Management](#service--daemon-management)
- [Software Package Management](#software-package-management)
  - [What is a Package?](#what-is-a-package)
  - [What is Dependency Hell?](#what-is-dependency-hell)
  - [Package Managers by Distribution](#package-managers-by-distribution)
  - [Package Naming Conventions](#package-naming-conventions)
  - [Working with `dpkg` (Debian family)](#working-with-dpkg-debian-family)
  - [Using `apt`](#using-apt)
  - [apt vs apt-get](#apt-vs-apt-get)
  - [Universal Package Managers](#universal-package-managers)

---

## Service / Daemon Management

A **service** (or **daemon**) is a background process. A classic example is the **`ssh`** service, which allows remote connections to devices.

### Listing available services
Navigate to the init scripts directory and list its contents:

```bash
cd /etc/init.d
ls
```

### Checking a service's file type
`ssh` is actually a **shell script**. You can verify this with:

```bash
file ssh
```

### Managing services (classic init scripts)

```bash
./ssh start
./ssh stop
./ssh status
./ssh reload
```

### Managing services with `systemd` (modern distros)

```bash
systemctl status ssh
systemctl start ssh
# and so on (stop, restart, reload, enable, disable)
```

---

## Software Package Management

### What is a Package?

Think of a package as a carefully prepared **"software gift box"**. It is not just a single executable file, but a complete installation unit containing everything needed to get the software running smoothly.

Inside this "box" typically includes:

- **Executable programs**: The software body you actually run.
- **Libraries**: Shared code files that the software depends on to function.
- **Configuration files**: Settings files needed for the software to operate.
- **Metadata**: The software's name, version, and **dependencies** (which other packages it requires).
- **Installation scripts**: Instructions telling the system how to install, configure, or remove the software.

### What is Dependency Hell?

Dependency hell is a "chain reaction" problem caused by dependencies between software. Modern software rarely works in isolation — program A needs library B to run, B needs C, and C needs a specific version of D. When this dependency network becomes complex and conflicting, dependency hell occurs.

### Package Managers by Distribution

| Distribution Family | Examples | Package Format | Package Manager |
|---|---|---|---|
| Red Hat family | Fedora, Oracle Linux, Amazon Linux | RPM (Red Hat Package Manager) | **dnf** |
| Debian family | Ubuntu, Mint | DEB | **dpkg** / **apt** |
| SUSE family | openSUSE | — | **zypper** (zypp) |
| Alpine | Alpine Linux | — | **apk** |

### Package Naming Conventions

**RPM format:**
```
<package_name>-<version_num>-<release_num>.<distribution>.<architecture>.rpm
```

**DEB format:**
```
<package_name>_<version>_<architecture>.deb
```

**Example:**
```
nsnake_3.0.1-2.1_amd64.deb
```

---

### Working with `dpkg` (Debian family)

#### List installed packages
```bash
dpkg -l
```

#### Show details about a package
```bash
dpkg -p packageName
# example
dpkg -p bash
```

#### Download a package manually
Copy the package link from the Ubuntu Packages page, then download it with `wget`:

```bash
wget packageLink
```

#### Show details about an installed package
```bash
dpkg -I packageName
```

#### Install a downloaded package
```bash
sudo dpkg -i packageName
# example
sudo dpkg -i nsnake_3.0.1-2.1_amd64.deb
```

#### Show where a package is installed
```bash
dpkg -L packageName
# example
dpkg -L nsnake
```

#### Run the installed program
Use the name listed in `dpkg -l`:

```bash
nsnake
```

#### Remove an installed package
```bash
sudo dpkg -r nsnake
```

#### Find which package owns a file
```bash
dpkg -S fileName/filePath
```

---

### Using `apt`

Instead of manually searching for a package, downloading it with `wget`, then installing with `dpkg -i`, **`apt`** simplifies the entire process.

#### Install a package
```bash
sudo apt install nsnake
# or shorthand
sudo apt -i nsnake
```

#### Package sources
By default, `apt` searches the Ubuntu repositories. The **`sources.list`** file lets you specify which sources to search for packages.

On modern Ubuntu, you'll find it as `ubuntu.sources` in:
```
/etc/apt/sources.list.d/ubuntu.sources
```

#### Filtering sources with `grep`
```bash
cat sources.list | grep \#    # list all lines starting with #
cat sources.list | grep ^#    # list all lines starting with a single #
cat sources.list | grep -v ^# # list all lines that do NOT start with #
```
> The `-v` option inverts the match.

#### Update the package index
```bash
sudo apt update
```
This checks all installed package versions against the versions on the repositories and tells you how many packages can be upgraded. Then:
```bash
apt list --upgradable   # see which packages can be upgraded
```

#### Upgrade packages
```bash
sudo apt upgrade        # upgrade packages
sudo apt full-upgrade   # upgrade + remove obsolete packages
```
> **Difference:** `full-upgrade` will also remove packages that are no longer important/needed.

#### Search for packages
```bash
apt search packageName
```

#### Search only installed packages
```bash
apt search nsnake | grep "installed"
```

#### Remove a package
```bash
sudo apt remove packageName
```

---

### apt vs apt-get

**`apt-get`** is an older, more advanced low-level tool with more features and options than `apt`. Some operations are only possible with `apt-get`.

---

### Universal Package Managers

**Universal package managers / installers** are not distro-specific and work across distributions.

#### Snap
- Like an app store.
- Website: [snapcraft.io](https://snapcraft.io)
- Update all snap packages:
```bash
sudo snap refresh
```

#### Flatpak
Another universal installer.

#### AppImage
The concept of AppImage is to bundle the entire application into **one file**. Run that file, and the application runs — no installation required.

---

*Notes compiled for reference. Feel free to contribute or correct.*