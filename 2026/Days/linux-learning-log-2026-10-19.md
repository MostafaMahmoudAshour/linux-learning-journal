# Linux Basics — 2026-10-19

## Basic commands

| Command | What it does |
|---|---|
| `ls` | Lists files and directories in the current directory. |
| `ls -a` | Includes hidden entries (names beginning with `.`). |
| `ls -l` | Shows a long listing with details such as permissions, owner, size, and timestamp. |
| `ls -F` | Adds a character to indicate file types; directories end in `/`. |
| `ls -alF` | Combines the options above. Options can often be combined, e.g. `ls -al`. |
| `pwd` | Prints the current working directory's full path. |
| `clear` or `Ctrl + L` | Clears the terminal display. |
| `echo "message"` | Prints text. |
| `history` | Displays commands recorded in shell history. |
| `ip address` or `ip a` | Shows network interface and IP-address information. |
| `man commandName` | Opens the manual page for a command. |
| `commandName --help` | Shows command usage and available options when supported. |
| `type -a commandName` | Shows how the shell resolves a command, including aliases, functions, built-ins, and executable paths. |

## Files and directories

| Command | What it does |
|---|---|
| `mkdir directoryName` | Creates a directory. |
| `mkdir -p my_dir && cd my_dir` | Creates the directory (including missing parent directories) and enters it if creation succeeds. |
| `rmdir directoryName` | Removes an empty directory. |
| `rm fileName` | Removes a file. |
| `rm -r directoryName` | Removes a directory and its contents recursively. **Use carefully:** this can permanently delete data. |
| `touch fileName` | Creates an empty file if it does not exist; otherwise, updates its timestamps. |
| `cat fileName` | Prints a file's contents; it can also concatenate multiple files. |

## Paths and directories

- `/` is the root directory of the Linux filesystem hierarchy. It is not the same as a Windows drive such as `C:`.
- `.` refers to the current directory.
- `..` refers to the parent directory.
- `~` is shorthand for the current user's home directory, commonly `/home/username` on Ubuntu.
- These commands take you to your home directory:
  - `cd`
  - `cd ~`
  - `cd /home/username` (replace `username` with the actual username).
- Linux paths use forward slashes (`/`), while Windows paths commonly use backslashes (`\\`).

Example: if the current directory is `/home/Mostafa/Documents`, then `.` means `Documents` and `..` means `Mostafa`.

## Hidden files and terminal colors

- Names beginning with a dot, such as `.bashrc`, are hidden by default in ordinary `ls` output. Use `ls -a` to show them.
- Common terminal color schemes show directories in blue and executable files in green, but colors depend on the terminal and `ls` configuration.
- Linux executables do not need a `.exe` extension.
- `ls` may be an alias such as `ls --color=auto`. Running `/bin/ls` or `/usr/bin/ls` invokes the executable directly and may bypass the alias. Use `/bin/ls --color=auto` to request colors explicitly.

## Shell built-ins, executables, and aliases

Some commands are built into the shell and do not require a separate executable for the shell to run them. Examples include `cd` and `bg` in Bash. Other commands, such as `ls`, are usually external programs.

```bash
type -a ls
type -a cd
```

`type -a` can show aliases, shell built-ins, functions, and executable locations. Therefore, `ls`, `/bin/ls`, and `/usr/bin/ls` may run the same underlying program, but the first can be affected by an alias.

## Shell history and variables

```bash
echo "$HISTSIZE"
echo "$HISTFILESIZE"
history
```

- `HISTSIZE` controls how many commands Bash keeps in its in-memory history list for the current shell.
- `HISTFILESIZE` controls the maximum number of lines kept in the history file, when Bash writes history to it.
- The history file is commonly `~/.bash_history` in Bash, though settings can differ.
- Double quotes allow shell variable expansion: `echo "$HISTSIZE"` prints the variable's value.
- Single quotes preserve the text literally: `echo '$HISTSIZE'` prints `$HISTSIZE`.

## Root privileges and package installation

- `root` is the superuser account with broad administrative privileges.
- `sudo` runs a command with elevated privileges when the user is authorized.
- Example on Ubuntu/Debian:

```bash
sudo apt install vim
```

This installs the `vim` text editor using the APT package manager, with administrative privileges.

## Case sensitivity

Linux commands, paths, and filenames are generally case-sensitive. For example, `Documents` and `documents` can be different directory names.

## Quick reminders

- Use `ls -alF` for a detailed listing that includes hidden entries and marks file types.
- Use `pwd` to check where you are before running file-removal commands.
- Be cautious with `rm -r`; double-check the target path before executing it.
- Use `man ls` or `ls --help` to learn more about a command.
