```bash
#!/bin/bash

# 1. Ensure Docker is installed
if ! command -v docker &> /dev/null; then
    command -v curl &> /dev/null || sudo apt install curl -y
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
fi

# 2. Enable Docker service
if ! systemctl is-active --quiet docker; then
    sudo systemctl start docker
    sudo systemctl enable docker
fi

# 3. Remove any running MySQL container
container_id=$(sudo docker container ls --filter "ancestor=mysql" -q)
[[ -n $container_id ]] && sudo docker rm $container_id -f

# 4. Run MySQL container
sudo docker run --rm --name mysql-db -h mysql-db \
    -e MYSQL_ROOT_PASSWORD=P@ss0rd -d mysql:latest

# 5. Wait for MySQL to be ready
for (( attempt=1; attempt<=9; attempt++ )); do
    if sudo docker exec -it mysql-db mysql -uroot -pP@ss0rd -e "SELECT 1;" &> /dev/null; then
        echo "Connected successfully"; break
    fi
    sleep 5
done

# 6. Create DB + table
sudo docker exec -i mysql-db mysql -w -uroot -pP@ss0rd -e \
    'CREATE DATABASE namesdb; USE namesdb;
     CREATE TABLE passwd (name VARCHAR(20), uid INT, gid INT,
       comments VARCHAR(30), home_dir VARCHAR(30), default_shell VARCHAR(30));'

# 7. Extract fields from /etc/passwd and load into arrays
_username=$(cut -d: -f1 /etc/passwd)
_uids=$(cut -d: -f3 /etc/passwd)
_gids=$(cut -d: -f4 /etc/passwd)
_comments=$(cut -d: -f5 /etc/passwd)
_home_dirs=$(cut -d: -f6 /etc/passwd)
_default_shells=$(cut -d: -f7 /etc/passwd)

declare -a usernames uids gids comments home_dirs default_shells

n=0; for name in $_usernames;     do usernames[n]=$name;        ((n++)); done
n=0; for uid in $_uids;           do uids[n]=$uid;              ((n++)); done
n=0; for gid in $_gids;           do gids[n]=$gid;              ((n++)); done
n=0; for c in $_comments;         do comments[n]=$c;            ((n++)); done
n=0; for hd in $_home_dirs;       do home_dirs[n]=$hd;          ((n++)); done
n=0; for ds in $_default_shells;  do default_shells[n]=$ds;     ((n++)); done

# 8. Insert into MySQL
for ((i=0;i<$n;i++)); do
    sudo docker exec mysql-db mysql -w --show-warnings=0 -uroot -pP@ss0rd -e \
        "USE namesdb; INSERT INTO passwd VALUES(
          '${usernames[i]}','${uids[i]}','${gids[i]}',
          '${comments[i]}','${home_dirs[i]}','${default_shells[i]}')" 2>/dev/null
done

echo "Data loaded successfully!"
```

# Line-by-Line Explanation of the ETL Script

This script is a **simple ETL (Extract → Transform → Load)** pipeline:

1. **Extract** user records from `/etc/passwd`
2. **Transform** them into arrays (one per field)
3. **Load** them into a MySQL table running inside Docker

Below is a breakdown of the interesting syntax and what each part does.

---

## Part 1 — Docker Installation Check

```bash
if ! command -v docker &> /dev/null; then
    command -v curl &> /dev/null || sudo apt install curl -y
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
fi
```

### `command -v docker`
- Returns the **path** of `docker` if it exists in `$PATH`, or nothing if not.
- Used to check "is this program installed?" — cleaner than `which` (which is not POSIX and not always present).
- Exit code: `0` if found, `1` if not.

### `&> /dev/null`
- `&>` redirects **both stdout (fd 1) and stderr (fd 2)** to a file.
- `/dev/null` is a special "black hole" file — anything written is discarded.
- So `command -v docker &> /dev/null` = "run the check **silently**, we only care about the exit code".

| Redirection | Meaning |
|-------------|---------|
| `>` | stdout only |
| `2>` | stderr only |
| `&>` | both stdout + stderr (Bash-specific) |
| `&> /dev/null` | discard everything |

### `!` (logical NOT)
- Inverts the exit code: succeeds if the command **fails**.
- `if ! command -v docker; then` = "if docker is **not** installed".

### `||` (OR)
```bash
command -v curl &> /dev/null || sudo apt install curl -y
```
- Runs `sudo apt install curl` **only if** the check for curl failed.
- This is the idiomatic "install if missing" one-liner.

### `curl -fsSL <url> -o get-docker.sh`
| Flag | Meaning |
|------|---------|
| `-f` | Fail silently on HTTP errors (no HTML error page saved) |
| `-s` | Silent (no progress bar) |
| `-S` | But still show errors |
| `-L` | Follow redirects |
| `-o file` | Write output to a file |

Common combo: **`-fsSL`** = silent + fail-on-error + show-errors + follow redirects.

### `sudo sh get-docker.sh`
- Runs the downloaded installer script with `sh` (not `bash`) — safer, doesn't depend on bash.

---

## Part 2 — Docker Service Check

```bash
if ! systemctl is-active --quiet docker; then
    sudo systemctl start docker
    sudo systemctl enable docker
fi
```

### `systemctl is-active --quiet docker`
- Prints `active`/`inactive` and **exits 0 if active**.
- `--quiet` suppresses the printed word — you only get the exit code.
- `enable` = auto-start at boot.
- `start` = start now.

---

## Part 3 — Kill Any Existing MySQL Container

```bash
container_id=$(sudo docker container ls --filter "ancestor=mysql" -q)
[[ -n $container_id ]] && sudo docker rm $container_id -f
```

### `$( ... )` — command substitution
- Captures the stdout of the command into the variable.
- `container_id` will hold a container ID (or be empty).

### `docker container ls --filter "ancestor=mysql" -q`
- `ls` = list **running** containers.
- `--filter "ancestor=mysql"` = only containers whose image is `mysql` (any tag).
- `-q` = **quiet** → print only container IDs (no table header).

### `[[ -n $container_id ]]`
- `-n` = "string is **n**on-empty".
- True if at least one container was found.

### `&&`
- Runs `docker rm ... -f` only if the test succeeded.
- `-f` = force (kills it if running).

> Contrast with `-z` = "string is **z**ero (empty)".

---

## Part 4 — Run MySQL Container

```bash
sudo docker run --rm --name mysql-db -h mysql-db \
    -e MYSQL_ROOT_PASSWORD=P@ss0rd -d mysql:latest
```

| Flag | Meaning |
|------|---------|
| `--rm` | Delete the container when it stops |
| `--name mysql-db` | Fixed container name (used later in `docker exec`) |
| `-h mysql-db` | Container hostname |
| `-e VAR=value` | Set an environment variable (here the root password) |
| `-d` | Detach (run in background) |
| `mysql:latest` | Image name:tag |

> Note: `mysql-db` is used as `-h` so clients inside the container can resolve it. Later commands use `mysql-db` as the **container name** in `docker exec`.

---

## Part 5 — Wait for MySQL to Be Ready

```bash
for (( attempt=1; attempt<=9; attempt++ )); do
    if sudo docker exec -it mysql-db mysql -uroot -pP@ss0rd -e "SELECT 1;" &> /dev/null; then
        echo "Connected successfully"; break
    fi
    sleep 5
done
```

### C-style `for (( ... ))`
- Same as in C: init, condition, increment.
- Runs up to **9 attempts**.

### `docker exec -it mysql-db mysql ...`
| Flag | Meaning |
|------|---------|
| `exec` | Run a command **inside** a running container |
| `-i` | Keep STDIN open (interactive) |
| `-t` | Allocate a TTY |
| `-it` | Combine both — needed for interactive shell; **not needed here** since we're not interactive. Should be `-i` only. |

- `mysql -uroot -pP@ss0rd -e "SELECT 1;"` connects and runs a trivial query.

### `&> /dev/null`
- Silence output — only the exit code matters.
- If `SELECT 1;` succeeds → exit 0 → **connected**, `break` out of the loop.

### `break`
- Jumps out of the loop immediately.

> **Bug:** The loop doesn't exit early if it exhausts attempts without success — the script just continues and later steps will fail. A more robust version would `exit 1` after the loop.

> **Security note:** Passing `-pP@ss0rd` on the command line **exposes the password** in `ps`, logs, and shell history.

---

## Part 6 — Create DB and Table

```bash
sudo docker exec -i mysql-db mysql -w -uroot -pP@ss0rd -e \
    'CREATE DATABASE namesdb; USE namesdb;
     CREATE TABLE passwd (...);'
```

### Multi-line `-e`
- `-e` accepts a full SQL string; multiple statements separated by `;`.
- Line continuation `\` lets the command span lines.

### `-w`
- `-w` (for `mysql`) = prompt for password before executing. **But** since we already supplied `-pP@ss0rd`, `-w` is redundant/confusing.

### Table
`passwd` mirrors the `/etc/passwd` structure:
| Column | Source field in `/etc/passwd` |
|--------|-------------------------------|
| `name` | field 1 |
| `uid` | field 3 |
| `gid` | field 4 |
| `comments` | field 5 |
| `home_dir` | field 6 |
| `default_shell` | field 7 |

---

## Part 7 — Extract Fields with `cut`

```bash
_username=$(cut -d: -f1 /etc/passwd)
```

### `cut -d: -f1`
- `-d:` = **delimiter is `:`** (default is TAB).
- `-f1` = take **field 1** (1-indexed).
- `/etc/passwd` looks like: `root:x:0:0:root:/root:/bin/bash`
- So `cut -d: -f1` returns `root` (the username).

| Field | Content |
|-------|---------|
| 1 | username |
| 2 | password placeholder (usually `x`) |
| 3 | UID |
| 4 | GID |
| 5 | comment (full name / GECOS) |
| 6 | home directory |
| 7 | login shell |

Fields **2** is skipped intentionally — not useful here.

### Why command substitution works here
- `$(cut ...)` returns a multi-line string.
- When you loop `for name in $_usernames`, Bash **word-splits** on whitespace (including newlines), so each line becomes one iteration.
- **Caveat:** this breaks if any field contains spaces — e.g. a user comment like "John Doe" would be split into two entries, misaligning arrays.

---

## Part 8 — Build Arrays

```bash
declare -a usernames uids gids comments home_dirs default_shells

n=0; for name in $_usernames; do usernames[n]=$name; ((n++)); done
```

### `declare -a ...`
- Declares **indexed arrays**. Bash doesn't strictly require it for assignment, but it's explicit and safe.

### `usernames[n]=$name`
- Stores value at index `n`.

### `((n++))`
- Arithmetic increment (C-style). Equivalent to `n=$((n+1))`.
- Notice each field has its **own** `n=0; for ...` block — so all arrays stay aligned index-by-index.

### Why this is fragile
The `for x in $string` pattern relies on **word splitting**. Two problems:
1. Comments like `"John Doe"` split into 2 iterations → all subsequent assignments shift → corruption.
2. Empty fields (`::` in passwd) **disappear** in word splitting → arrays become shorter than the real record count.

The robust pattern would be:
```bash
while IFS=: read -r name _ uid gid comment home shell; do
    usernames+=("$name")
    uids+=("$uid")
    ...
done < /etc/passwd
```
This avoids `cut` entirely and preserves empty fields.

---

## Part 9 — Insert Rows

```bash
for ((i=0;i<$n;i++)); do
    sudo docker exec mysql-db mysql -w --show-warnings=0 -uroot -pP@ss0rd -e \
        "USE namesdb; INSERT INTO passwd VALUES(
          '${usernames[i]}','${uids[i]}','${gids[i]}',
          '${comments[i]}','${home_dirs[i]}','${default_shells[i]}')" 2>/dev/null
done
```

- C-style loop over index `i`.
- `${array[i]}` = element at index `i` (note: no `$` inside `[]` — the whole `${...}` is one expansion).
- `--show-warnings=0` suppresses warnings.
- `2>/dev/null` discards **only stderr** (any error messages), so the script doesn't spew output.
- Each row is inserted with a fresh `docker exec` — **slow** (N execs). In production you'd batch with a single SQL file or `LOAD DATA INFILE`.

### SQL injection caveat
Values are interpolated into SQL with single quotes. If a user comment contained a `'`, the query would break (and be injectable). Passing data via `mysql < file` or a prepared statement avoids this.

---

## Quick Reference Table

| Syntax | What it does |
|--------|--------------|
| `cmd &> file` | Redirect stdout + stderr to file |
| `cmd 2>/dev/null` | Discard stderr only |
| `cmd >/dev/null 2>&1` | POSIX way to discard everything |
| `cmd1 \|\| cmd2` | Run `cmd2` only if `cmd1` fails |
| `cmd1 && cmd2` | Run `cmd2` only if `cmd1` succeeds |
| `! cmd` | Invert exit code |
| `$(...)` | Capture stdout into variable |
| `[[ -n $var ]]` | Non-empty string test |
| `[[ -z $var ]]` | Empty string test |
| `declare -a arr` | Declare indexed array |
| `(( expr ))` | Arithmetic evaluation |
| `cut -d: -fN` | Cut field N using `:` delimiter |
| `command -v prog` | Test if `prog` is in `$PATH` |
| `systemctl is-active --quiet svc` | Test if service is running (silent) |
| `docker exec -i ctr cmd` | Run `cmd` inside running container `ctr` |
| `docker container ls --filter "ancestor=img" -q` | List running containers using image, IDs only |

---

## Summary of What Could Be Improved

1. **`docker exec -it` in the readiness check** — `-t` isn't needed and can misbehave in scripts; use `-i` only (or none).
2. **Password on the command line** — leak risk; use a `.my.cnf` or env file.
3. **`while IFS= read`** should replace the `cut` + `for-in` pattern to preserve empty fields and spaces.
4. **Loop doesn't fail if MySQL never becomes ready** — add `exit 1` after the retry loop.
5. **One `docker exec` per row is slow** — use a single SQL file.
6. **SQL injection via user comment** — quote-escape values or use a file-based loader.
7. **Homework** from your notes: add distro detection (Debian vs Fedora) before Docker installation — `apt` vs `dnf`, `systemctl` paths, etc.

---

# Modified Vesion Of The Script

```bash
#!/bin/bash
set -euo pipefail

# ============================================================
# 0. Distribution Detection
# ============================================================
detect_distro() {
    if [[ -f /etc/os-release ]]; then
        . /etc/os-release
        DISTRO_ID="${ID:-unknown}"
        DISTRO_LIKE="${ID_LIKE:-}"
        DISTRO_NAME="${PRETTY_NAME:-$DISTRO_ID}"
    elif [[ -f /usr/lib/os-release ]]; then
        . /usr/lib/os-release
        DISTRO_ID="${ID:-unknown}"
        DISTRO_LIKE="${ID_LIKE:-}"
        DISTRO_NAME="${PRETTY_NAME:-$DISTRO_ID}"
    else
        echo "Error: unable to detect distribution (no os-release file)" >&2
        exit 1
    fi

    # Determine package manager family
    if [[ "$DISTRO_ID" == "debian" || "$DISTRO_ID" == "ubuntu" || "$DISTRO_LIKE" == *"debian"* || "$DISTRO_LIKE" == *"ubuntu"* ]]; then
        PKG_FAMILY="debian"
    elif [[ "$DISTRO_ID" == "fedora" || "$DISTRO_ID" == "rhel" || "$DISTRO_ID" == "centos" || "$DISTRO_LIKE" == *"fedora"* || "$DISTRO_LIKE" == *"rhel"* ]]; then
        PKG_FAMILY="fedora"
    else
        echo "Error: unsupported distribution: $DISTRO_NAME" >&2
        exit 1
    fi

    echo "Detected distribution: $DISTRO_NAME (family: $PKG_FAMILY)"
}

install_docker_debian() {
    echo "Installing Docker from official apt repository..."
    sudo apt update
    sudo apt install -y ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add repository (supports both old and new source formats)
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
}

install_docker_fedora() {
    echo "Installing Docker from official dnf repository..."
    sudo dnf -y install dnf-plugins-core
    sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
    sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
}

# ============================================================
# 1. Install Docker (based on distribution)
# ============================================================
detect_distro

if ! command -v docker &> /dev/null; then
    if [[ "$PKG_FAMILY" == "debian" ]]; then
        install_docker_debian
    else
        install_docker_fedora
    fi
else
    echo "Docker is already installed."
fi

# ============================================================
# 2. Enable and start Docker
# ============================================================
if ! systemctl is-active --quiet docker; then
    sudo systemctl enable --now docker
fi

# ============================================================
# 3. Clean up existing MySQL container
# ============================================================
container_id=$(sudo docker container ls --filter "ancestor=mysql" -q)
if [[ -n "$container_id" ]]; then
    sudo docker rm -f "$container_id"
fi

# ============================================================
# 4. Start MySQL container
# ============================================================
sudo docker run --rm --name mysql-db -h mysql-db \
    -e MYSQL_ROOT_PASSWORD='P@ss0rd' \
    -d mysql:latest

# ============================================================
# 5. Wait for MySQL to be ready (fail fast)
# ============================================================
MYSQL_READY=0
for attempt in {1..9}; do
    if sudo docker exec -i mysql-db mysql -uroot -p'P@ss0rd' -e "SELECT 1;" &> /dev/null; then
        echo "MySQL is ready."
        MYSQL_READY=1
        break
    fi
    echo "Waiting for MySQL... ($attempt/9)"
    sleep 5
done

if [[ "$MYSQL_READY" -eq 0 ]]; then
    echo "Error: MySQL did not become ready in time." >&2
    sudo docker rm -f mysql-db 2>/dev/null || true
    exit 1
fi

# ============================================================
# 6. Create database and table
# ============================================================
sudo docker exec -i mysql-db mysql -uroot -p'P@ss0rd' <<'SQL'
CREATE DATABASE IF NOT EXISTS namesdb;
USE namesdb;
CREATE TABLE IF NOT EXISTS passwd (
    name VARCHAR(32),
    uid INT,
    gid INT,
    comments VARCHAR(64),
    home_dir VARCHAR(64),
    default_shell VARCHAR(64)
);
SQL

# ============================================================
# 7. Extract and load data (use while read, preserve empty fields)
# ============================================================
echo "Loading data from /etc/passwd..."

# Use a temporary SQL file to avoid one docker exec per row
TMP_SQL=$(mktemp)
trap 'rm -f "$TMP_SQL"' EXIT

while IFS=: read -r name _ uid gid comment home shell; do
    # Escape single quotes to prevent SQL injection / syntax errors
    name_esc="${name//\'/\'\'}"
    comment_esc="${comment//\'/\'\'}"
    home_esc="${home//\'/\'\'}"
    shell_esc="${shell//\'/\'\'}"

    cat >> "$TMP_SQL" <<SQL
INSERT INTO passwd VALUES ('$name_esc', $uid, $gid, '$comment_esc', '$home_esc', '$shell_esc');
SQL
done < /etc/passwd

# Execute all INSERTs in one connection
sudo docker exec -i mysql-db mysql -uroot -p'P@ss0rd' namesdb < "$TMP_SQL" 2>/dev/null || {
    echo "Error: data loading failed." >&2
    exit 1
}

echo "Data loaded successfully."
```

---
Key Changes Explained
1. Distribution detection and branching installation
This is the focus of the homework. Read the ID and ID_LIKE from /etc/os-release to determine the system belongs to the Debian family or Fedora family. Installation commands for the two families are completely different:

Debian/Ubuntu: Use apt + Docker's official GPG key + apt source

Fedora/RHEL/CentOS: Use dnf + dnf-plugins-core + Docker's dnf repository

It is no longer the original "download get-docker.sh and run it directly" approach. That script is convenient but not suitable for production environments. The official documentation explicitly states that installation via repository is the recommended method.

2. MySQL wait loop now exits on failure
Added MYSQL_READY flag. If all 9 attempts fail, it cleans up the container and exit 1. The original script would continue executing subsequent SQL after timeout, causing a series of errors.

3. Removed -t and -w
docker exec -it in the readiness check: -t allocates a TTY, which is unnecessary in scripts and may produce warnings in non-interactive environments. Changed to -i.

mysql -w: redundant when password is already provided via -p, removed.

4. Password handling improvement
-pP@ss0rd (no space) will expose the password in ps output and shell history. Changed to -p'P@ss0rd' (still on command line, but with quotes to prevent accidental splitting). A more thorough solution would be to use MYSQL_PWD environment variable or --defaults-extra-file, but that would increase complexity, so keeping it simple here.

5. ETL logic rewritten with while IFS=: read
The original approach used cut to extract all fields first, then used for x in $string to split by whitespace into arrays. This has two fatal flaws:

If the comments field contains spaces (such as John Doe), word splitting will misalign all subsequent arrays.

Empty fields (consecutive ::) are discarded during word splitting, causing array lengths to mismatch.

while IFS=: read -r name _ uid gid comment home shell directly reads one record per line, with _ skipping the password placeholder field, naturally preserving empty fields and containing spaces, with completely correct alignment.

6. Batch insertion instead of one docker exec per row
All INSERT statements are written to a temporary SQL file, then executed once via docker exec -i mysql-db mysql ... < "$TMP_SQL". The original approach started a docker exec and MySQL connection for each user, which is extremely inefficient with dozens of users.

7. SQL injection protection
The comment field in /etc/passwd may contain single quotes ('). The original string concatenation would break SQL. Added ${var//\'/\'\'} to escape single quotes to double single quotes, which is standard SQL escaping syntax.

8. Minor robustness
set -euo pipefail: exits on error, undefined variables, and pipeline errors.

CREATE DATABASE IF NOT EXISTS / CREATE TABLE IF NOT EXISTS: re-running won't error.

trap 'rm -f "$TMP_SQL"' EXIT: ensures temporary files are cleaned up even if the script is interrupted.

Still Not Addressed (for reference)
Non-interactive sudo: If the script runs in an environment without a TTY, sudo may prompt for a password and block. It is recommended to configure NOPASSWD or run the entire script as root.

Docker user group: The current script still uses sudo docker. If you want to run directly as a normal user, you need sudo usermod -aG docker $USER then log out and back in. This is mentioned in Docker's post-installation documentation, but it's beyond the scope of this article, so it's not included.