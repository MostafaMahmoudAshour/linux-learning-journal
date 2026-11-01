# Linux Notes: Bash Expansions, Variables, and Shells

**Date:** 2026-10-21\
**Topic:** Bash expansions, variables, arithmetic, useful built-ins, and
shell types

## 1. What are shell expansions?

Expansions are performed by the shell (such as Bash) before a command
executes. The shell replaces expressions with values or lists of values.

``` bash
echo "$HOME"
echo file{1..3}.txt
echo $((4 * 5))
```

Common expansion types:

  -----------------------------------------------------------------------
  Type                    Example                 Purpose
  ----------------------- ----------------------- -----------------------
  Tilde expansion         `~`                     Represents the home
                                                  directory

  Parameter expansion     `$HOME`, `${name}`      Reads a variable's
                                                  value

  Command substitution    `$(date)`               Inserts the output of a
                                                  command

  Arithmetic expansion    `$((4 + 5))`            Evaluates integer
                                                  arithmetic

  Brace expansion         `file{1,2,3}.txt`       Generates strings

  Filename expansion      `*.txt`, `file[1-3]`    Matches existing
  (globbing)                                      filenames
  -----------------------------------------------------------------------

### Brace expansion vs. filename expansion

Brace expansion generates text whether or not matching files exist:

``` bash
echo file{1..3}.txt
# file1.txt file2.txt file3.txt
```

Filename expansion matches existing filenames:

``` bash
ls *.txt
ls file[1-3]
ls file[0-9][0-9]
ls [[:digit:]]file
```

Useful glob patterns:

-   `*` --- zero or more characters.
-   `?` --- exactly one character.
-   `[abc]` --- one character from the set.
-   `[0-9]` --- one digit.
-   `[!2]` --- one character other than `2` in that position.
-   `[[:digit:]]` --- one digit.

Globbing is not the same as regular expressions. For example,
`file[0-9][0-9]` matches filenames with two digits after `file`.

Create multiple directories using brace expansion (the parent directory
must already exist):

``` bash
mkdir project/{data,src,tests}
```

> **Note:** `touch file[1..10]` is not the usual way to generate ten
> numbered files. Use `touch file{1..10}` for `file1` through `file10`.

## 2. Variables and parameter expansion

Assign values without spaces around `=`:

``` bash
var1=10
var2=test
name="Mostafa Mahmoud"

echo "$var1"
echo "$var2"
echo "${var2}"
echo "$name"
```

`${name}` is another way to write `$name` and is useful when combining a
variable with surrounding text:

``` bash
echo "Hello, ${name}!"
```

An unset variable normally expands to an empty string:

``` bash
echo "$var3"
```

Provide a fallback value if a variable is unset:

``` bash
echo "${var4-novariable}"
```

This prints the value of `var4` if it is set; otherwise, it prints
`novariable`. The `-` form checks whether the variable is unset, while
`:-` also treats an empty value as missing.

### Quoting matters

``` bash
echo "$HOME"       # expands the variable
echo '$HOME'       # prints the literal text $HOME
```

Double quotes preserve an expanded value as one argument. Single quotes
prevent expansions inside them. In general, use `"$variable"` unless you
specifically need word splitting or filename expansion.

## 3. Command substitution

`$(command)` runs a command and substitutes its output:

``` bash
var1=$(date)
echo "$var1"

current_dir=$(pwd)
echo "$current_dir"
```

Remember to use `$var1` to print the variable's value; `echo var1`
prints the word `var1`.

## 4. Arithmetic

### Bash arithmetic expansion

``` bash
echo $((4 * 5))    # 20
echo $((10 / 4))   # 2 (integer division)
```

Bash's `$((...))` arithmetic uses integers, so division drops the
fractional part.

### `declare -i`

The integer attribute lets Bash treat assignments as arithmetic
expressions:

``` bash
declare -i num1 num2
num1=4
num2=num1*5
echo "$num2"       # 20
```

### `expr`

`expr` is an older command-line expression tool. Put spaces between
operands and escape special characters when needed:

``` bash
expr 1 + 2
expr 6 / 3
expr 3 \* 4
expr 5 \> 6
expr 5 == 5
```

For `expr`, a true comparison typically returns `1` and a false
comparison returns `0`.

### `bc` for decimal arithmetic

`bc` supports decimal calculations and arbitrary precision. Set `scale`
to choose the number of decimal places:

``` bash
echo "scale=2; 5 / 4" | bc
# 1.25
```

The `-l` option loads the math library:

``` bash
echo "s(1)" | bc -l
```

Base conversions:

``` bash
echo "obase=16; 255" | bc   # FF
echo "ibase=2; 1101" | bc   # 13
```

Run `bc -l` by itself for interactive use; type `quit` to exit.

## 5. Useful shell variables

  -----------------------------------------------------------------------
  Variable                            Meaning
  ----------------------------------- -----------------------------------
  `$PATH`                             Directories searched for executable
                                      commands

  `$HOSTNAME`                         System hostname (when set in the
                                      environment)

  `$SHELL`                            User's configured login shell; not
                                      necessarily the currently running
                                      shell

  `$USER`                             Username (when set in the
                                      environment)

  `$HOME`                             Home directory

  `$PWD`                              Current working directory

  `$?`                                Exit status of the most recently
                                      executed command
  -----------------------------------------------------------------------

Exit status convention:

-   `0` means success.
-   A non-zero value usually indicates failure or another status.

Example:

``` bash
true
echo "$?"     # 0

false
echo "$?"     # non-zero (often 1)
```

Because `$?` is updated after each command, inspect it immediately after
the command you want to check.

## 6. Chaining commands

A semicolon runs commands sequentially regardless of whether the earlier
command succeeds:

``` bash
echo "$var1"; ls
false; echo "$?"
```

In the second example, `echo` prints the exit status of `false`.

## 7. Listing variables and completion candidates

### `set`

``` bash
set
set | grep var
```

`set` displays shell variables, functions, and shell settings; piping to
`grep` filters the output for lines containing `var`.

### `compgen`

`compgen` is a Bash built-in that lists completion candidates:

  Command        Lists
  -------------- ------------------------
  `compgen -c`   Command names
  `compgen -a`   Aliases
  `compgen -b`   Bash built-ins
  `compgen -k`   Bash reserved keywords
  `compgen -u`   Usernames
  `compgen -f`   Files
  `compgen -d`   Directories
  `compgen -v`   Shell variable names

## 8. Local shell variables vs. exported environment variables

A variable created in a Bash shell is available in that shell, but a
newly started child shell does not automatically inherit ordinary,
unexported shell variables.

``` bash
var1=10
bash
echo "$var1"     # usually empty in the child shell
exit
```

Export the variable to pass it to child processes:

``` bash
export var1
bash
echo "$var1"     # 10
exit
```

You can also assign and export in one step:

``` bash
export var1=10
```

Use this to list exported variables:

``` bash
declare -x
```

Exported variables are inherited by child processes; they do not
automatically become available to unrelated shells or parent processes.

## 9. `declare` attributes

Bash's `declare` can assign attributes to variables:

  Option   Meaning
  -------- --------------------------------------
  `-a`     Indexed array
  `-A`     Associative array
  `-i`     Arithmetic/integer attribute
  `-l`     Convert assigned values to lowercase
  `-n`     Name reference
  `-r`     Read-only variable
  `-u`     Convert assigned values to uppercase
  `-x`     Export variable to child processes

Examples:

``` bash
declare -u var5="mostafa"
echo "$var5"      # MOSTAFA

declare -i num1 num2
num1=4
num2=num1*5
echo "$num2"      # 20
```

## 10. Login vs. non-login shells

The distinction is about how a shell starts and which startup files it
reads.

-   **Login shell:** Starts as a login session and reads login startup
    configuration.
-   **Non-login shell:** Commonly starts from an existing session and
    reads different startup configuration.

In Bash, check `$0`:

``` bash
echo "$0"
```

When Bash is invoked as a login shell, its name commonly begins with a
hyphen, such as `-bash`; a non-login Bash commonly appears as `bash`.
This is a useful clue, though `$0` can vary depending on how the shell
was started.

------------------------------------------------------------------------

## Quick practice checklist

-   [ ] Explain what shell expansion means.
-   [ ] Distinguish brace expansion from filename expansion.
-   [ ] Use `$VARIABLE`, `${VARIABLE}`, and `$(command)`.
-   [ ] Explain why `"$variable"` differs from `'$variable'`.
-   [ ] Use `$((...))` for integer arithmetic and `bc` for decimal
    arithmetic.
-   [ ] Check a command's exit status with `$?`.
-   [ ] Explain local variables, `export`, and `declare -x`.
-   [ ] Use `compgen` to list completion candidates.
-   [ ] Distinguish login and non-login Bash shells.
