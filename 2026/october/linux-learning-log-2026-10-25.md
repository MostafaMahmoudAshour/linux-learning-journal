# Linux Shell Notes: Variables, Streams, Pipes & Filters

## 1. Persisting Variables

Variables are lost when the shell session closes. To make them **persistent**, store them in shell startup files.

### Startup Files

| File | Scope | Notes |
|------|-------|-------|
| `/etc/profile` | **Login shells only** | Executed every time you open a login shell. System-wide. |
| `~/.bashrc` | **Login + non-login shells** | User-specific. Common across Linux distros. |

Put variables in these files so they persist across shell sessions.

---

## 2. Data Streams & Standard Streams

- **Data streams**: data/commands received or produced by the command line. In a terminal, these are called **text data streams**.
- **Standard streams**: data streams related to the terminal, whether received from or produced by it (terminal ↔ shell).

### The Three Standard Streams

| Stream | Index | Direction | Description |
|--------|-------|-----------|-------------|
| **STDIN** (Standard Input) | `0` | Terminal → Shell | Commands/streams going from terminal to shell |
| **STDOUT** (Standard Output) | `1` | Shell → Terminal | Successful output (no errors) |
| **STDERR** (Standard Error) | `2` | Shell → Terminal | Output with errors (from command or otherwise) |

> **Token**: each command word written in the terminal. Tokens are separated by spaces or delimiters.
> Example: `ls -l ~` → **3 tokens**.
> The full input typed is **STDIN**; a successful result is **STDOUT** (verify with `echo $?` → `0`).

---

## 3. Redirection

Redirect output away from the terminal to a file or another destination.

| Symbol | Meaning |
|--------|---------|
| `>` | Redirect STDOUT (same as `1>`) — **overwrites** file |
| `>>` | Redirect STDOUT — **appends** to file |
| `2>` | Redirect STDERR |
| `<` | Redirect STDIN (rarely useful: `cat < file` ≡ `cat file`) |

### Examples

```bash
ls -l > output                  # save STDOUT to file (overwrite)
ls /err 2> error                # save STDERR to file
cat /etc/shadow /etc/hosts 1> success 2> error
# /etc/shadow → error file (needs permissions)
# /etc/hosts  → success file
```

---

## 4. Pipes

A pipe (`|`) sends the **output of one command as input to another**.

```bash
head -n 50 /var/log/syslog | wc -w
head -n 50 /var/log/syslog | grep -i snap
head -n 50 /var/log/syslog | grep -i snapc | wc -w

# Store piped result in a variable (useful in scripts)
result=$(head -n 50 /var/log/syslog | grep -i snapc | wc -w)
echo $result
```

---

## 5. Filters

Filters modify the style or form of text output.

### `grep` — search text in files

```bash
grep ff00 /etc/hosts          # print lines containing "ff00"
grep -n ff00 /etc/hosts       # -n : show line numbers
grep -r Port /etc/            # -r : recursive search
grep -rn Port /etc/           # combine flags
grep -i word file             # -i : ignore case
grep -v word file             # -v : invert (lines NOT containing word)
grep -c word file             # -c : count matching lines
```

> Variants: `fgrep`, `egrep`, etc.

### `head` / `tail` — first/last lines

```bash
head filePath                 # first 10 lines
head -n 20 filePath           # first 20 lines
tail filePath                 # last 10 lines
tail -n 20 filePath           # last 20 lines
```

### `wc` — word count

Output format: `numLines numWords numCharacters filePath`

```bash
wc -l /etc/hosts              # lines
wc -w /etc/hosts              # words
wc -c /etc/hosts              # characters
```

### `tr` — transform / translate

```bash
echo "This iss a test 123 !" | tr -d 's'          # delete all 's'
echo "This iss a test 123 !" | tr [i] [I]         # replace i → I
echo "This iss a test 123 !" | tr -s 's'          # squeeze repeated 's'
echo "This iss a test 123 !" | tr [a-z] [A-Z]     # lowercase → uppercase
echo "This iss a test 123 !" | tr -d [:digit:]    # delete digits
echo "This iss a test 123 !" | tr [:digit:] 'z'   # replace digits with z
cat /etc/hosts | tr [:lower:] [:upper:]           # uppercase file output
```

### `cut` — remove sections from each line

| Flag | Meaning |
|------|---------|
| `-c` | Cut by **character** |
| `-b` | Cut by **byte** |
| `-f` | Cut by **field** (requires `-d` delimiter) |
| `-d` | Specify **delimiter** |

```bash
echo "This is a string of bytes" | cut -c 1        # T
echo "This is a string of bytes" | cut -b 1        # T
echo "This is a string of bytes" | cut -c 3,9      # ia
echo "This is a string of bytes" | cut -c 5-10     # is a
echo "This is a string of bytes" | cut -c 5-10,15  # is a n

# Cut first field separated by space
echo "This is a string of bytes" | cut -f 1 -d " "   # This

# Cut fields from /etc/passwd (colon-delimited)
cat /etc/passwd | cut -f 1 -d ":"
cat /etc/passwd | cut -f 1,6 -d ":"
```

---

## Quick Reference

| Concept | Symbol / Command |
|---------|------------------|
| STDIN | `0` / `<` |
| STDOUT | `1` / `>` `>>` |
| STDERR | `2` / `2>` |
| Pipe | `\|` |
| Search | `grep` |
| First/last lines | `head` / `tail` |
| Count | `wc` |
| Translate | `tr` |
| Cut sections | `cut` |
| Persist vars | `/etc/profile`, `~/.bashrc` |