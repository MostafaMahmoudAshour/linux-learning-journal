# Linux Learning Notes — 2026-10-20

## 1. Editing Files with `vi` / `vim`

`vi filename` opens a file in **Normal (command) mode**. Press `Esc` to return to Normal mode from Insert mode.

### Essential commands

| Command | Action |
|---|---|
| `i` | Enter Insert mode before the cursor |
| `Esc` | Return to Normal mode |
| `h` / `j` / `k` / `l` | Move left / down / up / right |
| `w` | Move forward one word |
| `dw` | Delete from the cursor to the end of the word |
| `dd` | Delete the current line |
| `yy` | Copy (yank) the current line |
| `p` | Paste after the cursor / below the line |
| `o` | Open a new line below and enter Insert mode |
| `gg` | Go to the first line |
| `G` | Go to the last line |
| `ggdG` | Delete all lines in the file |

The `h`, `j`, `k`, and `l` movement keys are a longstanding `vi` convention.

### Save and quit

Run these commands from Normal mode:

| Command | Action |
|---|---|
| `:w` | Save the file |
| `:wq` or `:x` | Save and quit |
| `:q!` | Quit without saving changes |

### Run shell commands from `vi`

- `:!ls` — run `ls` without replacing text in the file.
- `:.!date` — replace the current line with the output of `date`.
- `:sh` — start a shell while leaving the editor open in the background. Type `exit` to return to `vi`.

**Note:** `:.!command` filters/replaces the selected text (here, the current line) with the command's output. Use `:!command` when you only want to run a command without replacing file content.

### Replace file contents from the terminal

```bash
echo "New content" > filename.txt
cat new_source.txt > filename.txt
> filename.txt
```

A single `>` **overwrites** the destination file; `> filename.txt` empties it. Use `>>` to append instead:

```bash
echo "Additional content" >> filename.txt
```

To write a line containing quotes safely:

```bash
echo 'echo "Hello world"' > file.sh
```

Copy files with:

```bash
cp source destination
```

## 2. Making a Script Executable

For a shell script, add execute permission and run it from the current directory:

```bash
chmod +x filename.sh
./filename.sh
```

- `chmod +x` adds execute permission.
- `./filename.sh` runs the script from the current directory.
- `chmod -x filename.sh` removes execute permission.
- `ls -l filename.sh` displays its permissions.

The `x` in the permission string indicates execute permission. Exact permissions vary with the file's existing permissions and system defaults.

## 3. `vi` and `nano`

`nano` is another terminal-based text editor and is often easier for beginners. Check whether it is available with:

```bash
which nano
```

If this prints a path, the command is available through the current `PATH`.

## 4. Terminal, TTY, Pseudo-Terminal, Shell, and Kernel

### How a command gets executed

A simplified modern flow is:

**Terminal emulator → pseudo-terminal (PTY) → shell → command/program → kernel and system resources → output back to the terminal**

- **Terminal emulator:** The window that displays text and accepts keyboard input.
- **TTY:** Historically a physical teletype terminal; the term is also used for terminal devices and sessions.
- **PTY (pseudo-terminal):** A software-based terminal pair used by terminal emulators and many remote sessions. A shell may be attached to a device such as `/dev/pts/0`.
- **Shell:** Interprets commands, handles built-ins, searches `PATH` for external programs, and starts programs.
- **Kernel:** Manages hardware and system resources and provides services that programs access through system calls.

The terminal emulator and shell are separate programs with different responsibilities.

### Inspect processes, shell, and terminal

```bash
ps
echo "$SHELL"
tty
```

- `ps` shows processes included in its default process listing.
- `echo "$SHELL"` displays the shell recorded in the `SHELL` environment variable; it may show your configured login shell rather than every detail of the current shell process.
- `tty` prints the terminal device connected to standard input, if there is one.

Opening another terminal window often creates another PTY, such as `/dev/pts/1`. The number depends on which sessions are already open.

### Virtual consoles and graphical sessions

Linux can provide multiple virtual consoles. On many Ubuntu installations, `Ctrl + Alt + F3` switches to a text login console, while `Ctrl + Alt + F2` may return to the graphical session. The exact function-key mapping varies by Ubuntu version and configuration.

A graphical desktop may contain multiple terminal-emulator windows, each with its own PTY. The desktop session and its terminal windows are not the same thing.

### SSH to the same machine

Install the OpenSSH server on Ubuntu with:

```bash
sudo apt install openssh-server
```

Then connect to the same computer over SSH:

```bash
ssh your_username@localhost
```

Replace `your_username` with your Linux username. If the SSH server is running and configured to accept the connection, this opens a shell session to the local machine. Type `exit` to end the session.

## 5. Common Shells

A shell is a command interpreter. Common examples include:

| Shell | Name / note | First released |
|---|---|---:|
| `sh` | Bourne shell; also used for compatible implementations | 1979 |
| `bash` | Bourne Again Shell; widely used on Linux | 1989 |
| `zsh` | Z shell; highly customizable | 1990 |
| `fish` | Friendly Interactive Shell; designed for interactive usability | 2005 |

Check the configured login shell:

```bash
echo "$SHELL"
```

List shell-related executable names in `/bin`:

```bash
ls /bin/*sh
```

The results depend on the distribution and installed software. To start a shell by typing `sh`, enter:

```bash
sh
```

Type `exit` to leave that shell and return to the previous one.

## 6. Second-System Syndrome and the Unix Philosophy

### Second-system syndrome

**Second-system syndrome** is the tendency to over-engineer a follow-up system by adding too many features and making it unnecessarily complex. The concept is associated with Fred Brooks's *The Mythical Man-Month* (1975).

Common causes include:

- Adding every feature postponed during the first version.
- Becoming overconfident after the first system succeeds.
- Expanding scope until the system becomes harder to maintain and deliver.

Avoid it by controlling scope, prioritizing essential features, and building incrementally. Examples discussed in the notes include IBM OS/360, Windows Vista, and Netscape 6.0.

### Multics, Unix, and Linux

- **Multics** was an ambitious time-sharing operating-system project involving MIT, General Electric, and Bell Labs.
- **Unix** was developed at Bell Labs by Ken Thompson, Dennis Ritchie, and others. It emphasized a smaller, modular design and influenced many later operating systems.
- **Linux**, created by Linus Torvalds in 1991, is a Unix-like kernel. Combined with system tools and other software, it forms part of many Linux distributions.

The relationship is best understood as influence and shared design ideas—not Linux being a direct copy of the original Unix source code.

### Nine ideas commonly associated with the Unix philosophy

1. Small is beautiful.
2. Make each program do one thing well.
3. Prototype as soon as possible.
4. Prefer portability where practical.
5. Store data in simple, reusable formats such as text when appropriate.
6. Use software leverage and reuse.
7. Use shell scripts to combine programs and automate tasks.
8. Avoid unnecessarily restrictive, captive interfaces.
9. Make programs work well as filters in pipelines.

**Core idea:** build small tools that do their jobs well and can be combined to solve larger problems.

## Quick Practice Checklist

- [ ] Create, edit, save, and quit a file with `vi`.
- [ ] Explain the difference between `:!date` and `:.!date`.
- [ ] Explain the difference between `>` and `>>`.
- [ ] Make a script executable with `chmod +x` and run it with `./`.
- [ ] Explain the roles of terminal emulator, PTY, shell, and kernel.
- [ ] Inspect processes and the current terminal with `ps` and `tty`.
- [ ] Open and exit a shell or SSH session.
- [ ] Summarize second-system syndrome and the Unix philosophy.
