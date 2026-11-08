# Shell/Bash Scripts Notes

## What is a Bash Script?
A text file containing Linux commands.

**Benefits:**
- Portability
- Scheduling

---

## Creating & Running a Script

```bash
nano first_script.sh        # create file
chmod +x first_script.sh    # make executable
./first_script.sh           # run (also: sh file.sh | bash file.sh)
```

> `./` is required because `first_script.sh` alone is treated as a command looked up in `$PATH`. Using `./` tells the shell it's a file in the current directory.  
> `PATH=.:$PATH` allows running without `./` but is **not recommended**.

---

## Shebang (`#!`)

Specifies the execution context (interpreter).

```bash
#!/bin/bash        # run with bash shell
#!/usr/bin/python3 # run with python3
#!/usr/bin/env python3  # use env to locate interpreter in PATH
```

---

## Command Chaining

| Operator | Meaning |
|----------|---------|
| `;` | Run commands sequentially |
| `&&` | Run second only if first succeeds |
| `\|\|` | Run second only if first fails |

```bash
echo "hello"; ls
echo "hello" && ls
false || ls   # runs ls
```

---

## Bash Kernel in VSCode/Jupyter

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install python3-pip -y
pip install bash_kernel
python3 -m bash_kernel.install
```
Then select **Bash** as kernel in a `.ipynb` file.

---

## Variables

Three types: **Integer, String, Constant**

```bash
declare -i sum=16              # integer
declare st="Some String"       # string
declare -r PI=3.14             # readonly (constant)
declare -a names=(Mostafa Mahmoud)  # indexed array
names[2]="Ashour"

echo $sum
echo ${names[0]}
exit 0
```

### `exit 0`
Explicitly signals success (`0` = success, `1–255` = failure). A convention that tells the calling process everything ran smoothly.

---

## Arithmetic Operations

```bash
echo $((num*5))     # $(())
expr $num + 6       # expr
echo $num ^ 4 | bc  # bc (bash calculator)
```

> **Note:** Stick to **one** method throughout a script.

---

## Logical Operations

```bash
true ; echo $?    # 0
false ; echo $?   # 1

[[ 3 == 3 ]]; echo $?
test 1 -le 5; echo $?

# File tests
test -f /etc/hosts ; echo $?   # file exists (-d for dir)
test -r /etc/shadow ; echo $?  # readable (-w write, -x execute)
```

> **Advice:** Use one style consistently; check official/modern resources.

---

## Positional Parameters

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$#` | Number of arguments |
| `$*` | List of arguments |
| `$@` | Like `$*` but as indexed array |
| `$1`, `$2` | First, second args |

```bash
echo "Args: $#"
echo "First arg: $1"
# ./test_script.sh Mostafa 22
```

---

## Shell Functions

```bash
Hello () {
    echo "Hello Functions!"
    timestmp
    return 0
}

function timestmp () {
    echo "The time is $(date +%m/%d/%y" "%R)"
}

Hello
```

---

## Reading from STDIN

```bash
read name
read -p "name: " name       # with prompt
read -sp "password: " pass  # secret input
```

---

## Loops

### `for` Loop
```bash
for name in Alice Bob; do echo "$name"; done
for i in {1..10..2}; do echo "$i"; done      # step
for ((i=0;i<5;i++)); do echo "$i"; done      # C-style
for file in *.txt; do echo "$file"; done     # globs
for fruit in "${fruits[@]}"; do echo "$fruit"; done  # array
for arg in "$@"; do echo "$arg"; done        # positional
```

### `while` Loop
```bash
count=1
while [ "$count" -le 5 ]; do ((count++)); done

while IFS= read -r line; do echo "$line"; done < file.txt

while true; do sleep 1; done
```

### `until` Loop
```bash
until [ "$count" -gt 5 ]; do ((count++)); done
```

### `select` Loop (interactive menu)
```bash
select option in "Start" "Stop" "Quit"; do
    case $option in
        Start) echo "Starting..." ;;
        Quit)  break ;;
    esac
done
```

### Loop Control
| Statement | Effect |
|-----------|--------|
| `break` | Exit loop |
| `break n` | Exit n nested levels |
| `continue` | Skip iteration |
| `continue n` | Skip to n-th outer iteration |

---

## Quick Comparison

| Loop | Use when |
|------|----------|
| `for item in list` | Known list of items |
| `for (( ; ; ))` | Numeric counter |
| `while` | Unknown iteration count |
| `until` | Run until condition true |
| `select` | Interactive menu |

---

## Conditional Expressions

```bash
if [ "foo" = "foo" ]; then
    echo "true"
fi
```

### Example: Even/Odd Checker (progressive refinement)

```bash
# Basic
read -p "Enter a number: " num
if (( num % 2 == 0)); then echo "$num is even"; fi

# With validation
if ! [[ "$num" =~ ^[0-9]+$ ]]; then
    echo "Error: Not an integer"; exit 1
else
    if (( $num % 2 == 0)); then echo "even"; else echo "odd"; fi
fi

# With function + loop
function is_int(){
    [[ "$num" =~ ^[0-9]+$ ]] && return 0 || return 1
}

while [[ ${num,,} != "quit" ]]; do
    read -p "Enter a number: " num
    if is_int; then
        (( num % 2 == 0 )) && echo "$num is even" || echo "$num is odd"
    else
        echo "Error: Not an integer"
    fi
done
```

---

## Useful String Tips

```bash
name=ahmed
echo ${name}      # ahmed
echo ${#name}     # 5 (length)
echo ${name^^}    # AHMED (uppercase)
echo ${name,,}    # ahmed (lowercase)
```

---

## Real Example: ETL Script

Pulls user data from `/etc/passwd` into a MySQL database running in Docker.

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

> **Homework:** Add a layer to detect the Linux distro, since Docker installation differs between Debian and Fedora.