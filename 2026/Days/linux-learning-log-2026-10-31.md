# Linux Process Management — Study Notes

## 1. What Is a Process?

A **process** is a running program. It happens when the OS reads the program file from disk, loads it into memory, and executes its instructions.

A process can run:
- In the **foreground** (attached to your terminal, you interact with it)
- In the **background** (running without occupying your terminal)

---

## 2. Types of Executable Programs That Become Processes

1. **C program** — compiled binary executed by the OS.
2. **Shell script** — its commands can be:
   - An **external program** (e.g., a C program)
   - A **built-in command** (installed by default with the shell)
3. **Single command** — can be:
   - **External** (has a path like `/bin/` or `/usr/sbin`)
   - **Built-in** shell command

---

## 3. Types of Processes

| Type | Description |
|------|-------------|
| **System process** | Executed by the kernel |
| **User process** | Executed by a user — manually, via SSH, or scheduled (e.g., cron) |

---

## 4. The Process Tree: systemd and PID 1

All processes descend from a single root process:

- **Old systems:** `SystemV init` (or just `init`)
- **Modern distros:** `systemd`

So:
- `systemd` is the **first process**, with **PID = 1**
- Every other process is a **child** branched from it
- The **parent of systemd is the kernel itself** (PID = 0)

---

## 5. Process Management Commands

```bash
ps          # show running processes for the current session/terminal and current user
ps -a       # all processes running from the current Linux user
ps -e       # details about kernel/system processes; systemd (PID=1) appears first
ps -l       # each process with details
ps -f       # processes with start time
ps -efl     # the most detailed listing

pstree      # show the tree of running processes

top         # live view of processes
htop        # advanced interactive process viewer

sleep 10    # delay for 10 seconds
sleep 1m    # delay for one minute; break with Ctrl + C

sleep 10 &  # run the command/process in the background
jobs        # list jobs running in the background
```

---

## 6. Key Notes

- If a process is run by the **kernel**, its **TTY = ?** (no controlling terminal).
- Any process running in the **background** — system or user — has a name ending in **`d`** (for **daemon**). Daemons are the Linux equivalent of **Windows services**.
- Every process has two IDs:
  - **PID** — Process ID
  - **PPID** — Parent Process ID
- To run a process in the background, append **`&`** to the command:
  ```bash
  sleep 5 &
  ```
  This prints a number in `[]` (the **job ID**) plus the **process ID**. Use `jobs` to list background jobs.
- **Ctrl + Z** stops (suspends) a process. Resume it with:
  - `fg` or `fg %jobNumber` — resume in the **foreground**
  - `bg` or `bg %jobNumber` — resume in the **background**

---

## 7. Signals and the `kill` Command

**Syntax:** `kill sigName/sigNumber PID`

| Signal | Number | Effect |
|--------|--------|--------|
| `SIGHUP` | `-1` | Restart the process with the same PID |
| `SIGINT` | `-2` | Same as **Ctrl + C** — ends the process and returns you to the terminal |
| `SIGTERM` | `-15` | Terminate **gracefully** — terminates child processes first, then the parent |
| `SIGKILL` | `-9` | **Brute-force** immediate kill; cannot be ignored. Recommended: try `SIGTERM` first, then `SIGKILL` |

**Kill by name:**
```bash
killall sigName/sigNumber processName
```
Example: kill all `vi` instances with `killall -9 vi`.

**Example:**
```bash
kill -15 4704
```

---

## 8. Why `kill -15` Sometimes Does Nothing (But `kill -9` Works)

**Core reason:** `kill -15` (**SIGTERM**) is only a **"polite request"** — the process can ignore or block it. `kill -9` (**SIGKILL**) is a **"forced execution order"** — the process has no ability to refuse.

### Why SIGTERM May Be Ineffective

SIGTERM is designed to let a process exit **gracefully**. It notifies the process "you should stop," but the process has the right to:

- **Ignore** the signal
- **Block** it (temporarily not process it)
- **Catch** it and run custom cleanup logic (e.g., saving data, closing files) instead of exiting immediately

If the program has a bug, is stuck in an **uninterruptible state**, or its developer simply never handled the signal, `kill -15` will appear to "do nothing." For example, some shell scripts or programs running in containers may not forward the signal to the actual working subprocess, causing SIGTERM to fail.

### Why SIGKILL Always Works

SIGKILL is enforced by the **OS kernel**. It **cannot be caught, blocked, or ignored** and terminates the process immediately. That's why it's the last resort when `kill -15` fails.

### Recommended Approach

1. **Try `kill -15` first** — gives the process a chance to save data and release resources safely.
2. **If it fails, use `kill -9`** — be aware this may cause data loss or leave residual temp files, so use it only as a last resort.