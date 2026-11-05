Here's your notes formatted as a clean markdown file ready to push to GitHub:


# SSH & Remote Server Notes

## Table of Contents
- [Connection to Remote Servers](#connection-to-remote-servers)
- [Connecting via IP / User](#connecting-via-ip--user)
- [SSH Config File (Aliases)](#ssh-config-file-aliases)
- [SSH Keys (Passwordless Login)](#ssh-keys-passwordless-login)
- [SSH Server Config](#ssh-server-config)
- [Copying Public Key with ssh-copy-id](#copying-public-key-with-ssh-copy-id)
- [Changing Key Algorithm](#changing-key-algorithm)
- [Transferring Files](#transferring-files)
- [Other Useful Tools](#other-useful-tools)
- [VS Code & Remote SSH](#vs-code--remote-ssh)
- [Open Questions](#open-questions)

---

## Connection to Remote Servers

- To connect to another remote machine/server shell, use the **SSH protocol** (open on port **22**).
- **SSH** stands for **Secure Shell**.
- You need a tool to use the protocol — just like `nginx` is used for HTTP/HTTPS.
- Most used tools:
  - **Linux → Windows:** `PuTTY`
  - **Linux → Linux:** `OpenSSH`
- To connect Linux → Linux, SSH must be installed on **both** client and server:
  - `openssh-client` (installed by default on most distros)
  - `openssh-server`
- To connect, provide the **username** and **password** — or use the **IP address** if credentials aren't available.
- Check if OpenSSH is installed:
  ```bash
  dpkg -l | grep openssh
  ```
- Get more info about the client package:
  ```bash
  apt show openssh-client
  ```


## Connecting via IP / User

```bash
# 1. Connect using IP address only
ssh 4.236.152.79

# 2. Connect using user@ip or user@hostname
ssh mostafa@4.236.152.79

# 3. Execute a command remotely and return the result (no shell access)
ssh mostafa@4.236.152.79 pwd
ssh mostafa@4.236.152.79 hostname   # returns the remote hostname
```

---

## SSH Config File (Aliases)

Configure aliases in `~/.ssh/config` (create it if it doesn't exist):

```bash
cd ~/.ssh
touch config
nano config
```

Example config:

```text
Host AzureLinux
    Hostname 4.236.152.79
    Port 22
    User mostafa
```

Now connect directly using the alias:

```bash
ssh AzureLinux
```

---

## SSH Keys (Passwordless Login)

- Every user should have **two keys**: **public** and **private**.
- Generate them with:
  ```bash
  ssh-keygen
  ```
- Copy the **public key** into the `authorized_keys` file on the remote machine.
- Next time you connect, SSH won't ask for a password.

---

## SSH Server Config

- View/modify SSH config:
  ```bash
  cd /etc/ssh
  nano sshd_config
  ```
- Commented lines = default values.

---

## Copying Public Key with ssh-copy-id

Alternative way to copy the public key to the remote machine:

```bash
ssh-copy-id -i public_key_file_name user@ipAddress
```

---

## Changing Key Algorithm

- Default algorithm for `ssh-keygen` is **RSA**.
- Change it with the `-t` flag:
  ```bash
  ssh-keygen -t algorithmName
  ```

> **TODO:** What is `ssh-agent`?

---

## Transferring Files

### `scp`

```bash
scp fileToMove remoteMachine:pathToMoveTo

# Example
scp file AzureLinux:/home/mostafa
```

### `rsync`

Another tool for transferring files (mentioned for further study).

---

## Other Useful Tools

- **`curl`** (Client URL): powerful tool supporting most protocols for sending files, making requests, and REST API calls.
- **`w3m`**: browse websites in the terminal, simulating the site's design in text form.

---

## VS Code & Remote SSH

- Install VS Code from the `.deb` file on the website:
  ```bash
  sudo dpkg -i fileName.deb
  ```
- Install the **Remote SSH** extension to open VS Code on a remote machine.

---

## Open Questions

- [ ] What is `ssh-agent`?
- [ ] Explore `rsync` in depth.

