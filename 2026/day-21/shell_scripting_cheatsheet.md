# Day 21: Shell Scripting Cheat Sheet

A practical Bash reference guide for Linux administration, log analysis, troubleshooting, and DevOps automation.

## Objectives

- Understand Bash scripting fundamentals.
- Use operators, conditionals, loops, and functions.
- Process text and analyze logs using Linux commands.
- Write safer scripts with error handling and debugging.
- Build reusable commands for everyday DevOps tasks.

---

## Quick Reference Table

| Topic | Key Syntax | Example |
|---|---|---|
| Shebang | `#!/bin/bash` | Defines the interpreter |
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Arguments | `$1`, `$2` | `./script.sh app.log` |
| If statement | `if [ condition ]; then` | `if [ -f "$file" ]; then` |
| For loop | `for i in list; do` | `for i in 1 2 3; do echo "$i"; done` |
| Function | `name() { ...; }` | `greet() { echo "Hi"; }` |
| Grep | `grep pattern file` | `grep -i "error" app.log` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed 's/foo/bar/g' config.txt` |
| Exit status | `$?` | `status=$?` |
| Strict mode | `set -Eeuo pipefail` | Enables common safety checks |

---

## Task 1: Basics

### 1. Shebang

The shebang specifies which interpreter should execute a script when it is run directly.

```bash
#!/bin/bash
echo "Hello, DevOps!"
```

Place the shebang on the first line of the script.

### 2. Running a Script

```bash
# Add execute permission
chmod +x script.sh

# Execute directly
./script.sh

# Execute through Bash
bash script.sh

# Check syntax without running
bash -n script.sh
```

- `chmod +x` adds execute permission.
- `./script.sh` runs the script from the current directory.
- `bash script.sh` explicitly invokes Bash.
- `bash -n` checks syntax without executing the script.

### 3. Comments

Comments explain code and are ignored by Bash.

```bash
# Single-line comment
name="Devesh"  # Inline comment

echo "Hello, $name"
```

### 4. Variables and Quoting

```bash
name="Devesh Arya"

echo "$name"   # Expands the variable and preserves spaces
echo '$name'   # Prints literal $name
echo $name     # May split words and expand wildcards
```

| Syntax | Meaning |
|---|---|
| `$VAR` | Expands a variable, but unquoted values may undergo word splitting and pathname expansion |
| `"$VAR"` | Expands a variable while preserving spaces as one argument in normal command contexts |
| `'$VAR'` | Prints literal text without expanding the variable |

**Best practice:** Quote variable expansions by default, for example, `"$file"`.

Do not put spaces around `=` when assigning variables.

```bash
name="Devesh"  # Correct
```

### 5. Reading User Input

The `read` command accepts input from the user.

```bash
read -r -p "Enter your name: " name
echo "Welcome, $name"
```

- `-r`: prevents backslashes from being interpreted specially.
- `-p`: displays a prompt.
- `-s`: hides typed input, useful for passwords.

Avoid printing or logging passwords.

### 6. Command-Line Arguments

```bash
#!/bin/bash

echo "Script: $0"
echo "First argument: ${1:-not provided}"
echo "Argument count: $#"

for arg in "$@"; do
    echo "Argument: $arg"
done

ls /invalid/path
status=$?

echo "Exit status: $status"
```

Run the script:

```bash
chmod +x args_demo.sh
./args_demo.sh alpha "beta gamma"
```

| Parameter | Meaning |
|---|---|
| `$0` | Script name or invocation path |
| `$1`, `$2` | First and second arguments |
| `$#` | Number of positional arguments |
| `"$@"` | All arguments, preserving each argument separately |
| `$?` | Exit status of the most recently completed command |

Usually, exit status `0` means success and a non-zero status means failure. Capture `$?` immediately because the next command changes its value.

---

## Task 2: Operators and Conditionals

### 1. String Comparisons

```bash
name="Devesh"

[ "$name" = "Devesh" ] && echo "Name matches"
[ "$name" != "Guest" ] && echo "Not a guest"
[ -z "$name" ] && echo "String is empty"
[ -n "$name" ] && echo "String is not empty"
```

| Operator | Meaning |
|---|---|
| `=` | Strings are equal |
| `!=` | Strings are different |
| `-z` | String is empty |
| `-n` | String is not empty |

### 2. Integer Comparisons

```bash
usage=85

if [ "$usage" -ge 80 ]; then
    echo "Warning: High usage"
fi
```

| Operator | Meaning |
|---|---|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-lt` | Less than |
| `-gt` | Greater than |
| `-le` | Less than or equal |
| `-ge` | Greater than or equal |

These operators compare integers, not arbitrary decimal numbers.

### 3. File Test Operators

```bash
file="app.log"

[ -f "$file" ] && echo "Regular file exists"
[ -d "backup" ] && echo "Directory exists"
[ -e "$file" ] && echo "Path exists"
[ -r "$file" ] && echo "Readable"
[ -w "$file" ] && echo "Writable"
[ -x "$file" ] && echo "Executable or searchable"
[ -s "$file" ] && echo "File is not empty"
```

| Operator | Purpose |
|---|---|
| `-f` | Path is an existing regular file |
| `-d` | Path is an existing directory |
| `-e` | Path exists |
| `-r` | Path is readable |
| `-w` | Path is writable |
| `-x` | File is executable or directory is searchable |
| `-s` | File exists and has a size greater than zero |

### 4. If, Elif, Else

```bash
usage=85

if [ "$usage" -ge 90 ]; then
    echo "CRITICAL: Disk usage"
elif [ "$usage" -ge 75 ]; then
    echo "WARNING: Disk usage"
else
    echo "Disk usage is normal"
fi
```

### 5. Logical Operators

```bash
[ -f "app.log" ] && echo "File exists"
[ -f "app.log" ] || echo "File is missing"
[ ! -f "app.log" ] && echo "No log file"
```

- `&&`: runs the next command if the previous command succeeds.
- `||`: runs the next command if the previous command fails.
- `!`: negates a condition.

### 6. Case Statements

Useful for menus and handling multiple choices.

```bash
read -r -p "Enter action: " action

case "$action" in
    start)  echo "Starting service" ;;
    stop)   echo "Stopping service" ;;
    status) echo "Checking service status" ;;
    *)      echo "Invalid option" ;;
esac
```

---

## Task 3: Loops

### 1. List-Based For Loop

```bash
for env in dev test prod; do
    echo "Environment: $env"
done
```

### 2. C-Style For Loop

```bash
for ((i = 1; i <= 5; i++)); do
    echo "Count: $i"
done
```

### 3. While Loop

Runs while its condition is true.

```bash
count=1

while [ "$count" -le 3 ]; do
    echo "$count"
    ((count += 1))
done
```

### 4. Until Loop

Runs until its condition becomes true.

```bash
count=1

until [ "$count" -gt 3 ]; do
    echo "$count"
    ((count += 1))
done
```

### 5. Break and Continue

```bash
for i in 1 2 3 4 5; do
    [ "$i" -eq 2 ] && continue
    [ "$i" -eq 4 ] && break
    echo "$i"
done
```

- `continue`: skips the current iteration.
- `break`: exits the loop.

### 6. Looping Over Files

```bash
for file in ./*.log; do
    [ -f "$file" ] || continue
    echo "Processing: $file"
    wc -l "$file"
done
```

The file check avoids processing the literal `./*.log` when no matching files exist.

### 7. Looping Over File Contents

```bash
while IFS= read -r line; do
    printf 'Line: %s\n' "$line"
done < app.log
```

Using input redirection keeps loop variable changes in the current shell.

---

## Task 4: Functions

### 1. Defining and Calling a Function

```bash
greet() {
    echo "Hello, DevOps!"
}

greet
```

### 2. Passing Arguments to Functions

```bash
show_info() {
    local file="$1"
    local label="$2"

    echo "$label: $file"
}

show_info "app.log" "Log file"
```

Inside a function, `$1` and `$2` refer to that function's arguments.

### 3. Return Values: `return` vs `echo`

`return` communicates a status code from `0` to `255`. Use `printf` or `echo` to produce text output.

```bash
is_readable() {
    [ -r "$1" ]
}

if is_readable "app.log"; then
    echo "File is readable"
fi
```

Capture text output:

```bash
get_greeting() {
    printf 'Hello, %s\n' "$1"
}

message=$(get_greeting "Devesh")
echo "$message"
```

### 4. Local Variables

```bash
greet() {
    local name="$1"
    echo "Hello, $name"
}

greet "Devesh"
```

Use `local` to limit a variable's scope to a function.

---

## Task 5: Text Processing Commands

Use practice files when testing commands that modify data.

### 1. Grep: Search Text

```bash
grep "ERROR" app.log
grep -i "error" app.log
grep -r "TODO" ./src
grep -c "ERROR" app.log
grep -n "ERROR" app.log
grep -v "INFO" app.log
grep -E "ERROR|Failed" app.log
```

| Flag | Purpose |
|---|---|
| `-i` | Ignore case |
| `-r` | Search recursively |
| `-c` | Count matching lines |
| `-n` | Display line numbers |
| `-v` | Invert the match |
| `-E` | Use extended regular expressions |

### 2. Awk: Process Fields and Patterns

```bash
awk '{print $1}' app.log
awk '{print $1, $2}' app.log
awk -F: '{print $1}' /etc/passwd
awk '/ERROR/ {print NR, $0}' app.log
awk 'BEGIN {print "Start"} /ERROR/ {count++} END {print count+0}' app.log
```

- `$1`, `$2`: first and second fields.
- `$0`: entire current line.
- `NR`: current record number.
- `-F`: sets the field separator.
- `BEGIN` and `END`: execute before and after input processing.

### 3. Sed: Stream Editing

```bash
sed 's/old/new/g' file.txt
sed '/DEBUG/d' app.log
sed -n '10,20p' app.log
sed -i.bak 's/foo/bar/g' config.txt
```

- `s/old/new/g`: replaces all occurrences per line.
- `/DEBUG/d`: excludes matching lines from output.
- `-n`: suppresses default output.
- `-i.bak`: edits the file and keeps a backup.

Test changes on copies before editing important configuration files.

### 4. Cut: Extract Fields

```bash
cut -d: -f1 /etc/passwd
cut -d, -f1,3 data.csv
cut -c1-10 app.log
```

- `-d`: delimiter.
- `-f`: field numbers.
- `-c`: character positions.

`cut` is suitable for simple delimited text but does not fully parse CSV fields containing quoted commas.

### 5. Sort

```bash
sort names.txt
sort -n numbers.txt
sort -r names.txt
sort -u names.txt
```

- Default: alphabetical sorting.
- `-n`: numeric sorting.
- `-r`: reverse order.
- `-u`: sort and remove duplicates.

### 6. Uniq

```bash
sort names.txt | uniq
sort names.txt | uniq -c
sort names.txt | uniq -d
```

- Default: remove adjacent duplicate lines.
- `-c`: count adjacent occurrences.
- `-d`: show duplicated lines.

Sort first when you want to count repeated values across the entire input.

### 7. Tr: Translate or Delete Characters

```bash
echo "hello" | tr 'a-z' 'A-Z'
echo "a,b,c" | tr ',' '\n'
echo "hello123" | tr -d '0-9'
```

### 8. Wc: Count Text

```bash
wc -l app.log
wc -w app.log
wc -c app.log
wc -m app.log
```

- `-l`: lines.
- `-w`: words.
- `-c`: bytes.
- `-m`: characters.

### 9. Head and Tail

```bash
head -n 10 app.log
tail -n 10 app.log
tail -f app.log
tail -F app.log
tail -F app.log | grep --line-buffered -E 'ERROR|CRITICAL'
```

- `head`: displays the beginning of a file.
- `tail`: displays the end of a file.
- `-f`: follows new data.
- `-F`: follows by filename and can handle log rotation.
- `--line-buffered`: displays matching lines promptly.

Press `Ctrl+C` to stop following a log.

---

## Task 6: Useful Patterns and One-Liners

**Safety note:** Test commands in a practice directory before deleting or modifying important files.

### 1. Find Files Older Than Seven Days

Preview matching files:

```bash
find ./logs -type f -name "*.log" -mtime +7 -print
```

Delete only after reviewing the results:

```bash
find ./logs -type f -name "*.log" -mtime +7 -delete
```

### 2. Count Lines in All Log Files

```bash
find ./logs -type f -name "*.log" -exec wc -l {} +
```

### 3. Replace Text and Keep a Backup

```bash
sed -i.bak 's/old_value/new_value/g' config.txt
```

### 4. Check Whether a Service Is Running

```bash
systemctl is-active nginx
```

With an explicit message:

```bash
if systemctl is-active --quiet nginx; then
    echo "Nginx is running"
else
    echo "Nginx is not active"
fi
```

### 5. Warn When Root Disk Usage Reaches 80%

```bash
df -P / | awk 'NR==2 {gsub(/%/, "", $5); if ($5 >= 80) print "WARNING: Root disk usage is " $5 "%"}'
```

This prints a warning. Sending email or SMS alerts requires a separate notification mechanism.

### 6. Count Lines Containing Errors

```bash
grep -Ec 'ERROR|Failed' app.log
```

### 7. Monitor Errors in Real Time

```bash
tail -F app.log | grep --line-buffered -E 'ERROR|CRITICAL|Failed'
```

### 8. Extract Simple CSV Columns

```bash
awk -F, '{print $1, $3}' transactions.csv
```

Use a CSV-aware parser for files with quoted commas.

### 9. Parse JSON with jq

```bash
jq '.name' config.json
jq -r '.users[].email' users.json
```

`jq` is safer than using `grep` to extract values from structured JSON. Install it if it is unavailable on your system.

---

## Task 7: Error Handling and Debugging

### 1. Exit Codes

```bash
exit 0  # Success
exit 1  # General failure
```

Check whether a pattern exists:

```bash
if grep -q "ERROR" app.log; then
    echo "Errors found"
else
    echo "No errors found"
fi
```

`grep` returns `0` for a match, `1` for no match, and another non-zero status for certain errors.

### 2. `set -e`

```bash
set -e
mkdir -p backup
```

Requests that Bash exit when certain commands fail. There are exceptions, so handle expected failures explicitly.

### 3. `set -u`

```bash
set -u
echo "$UNDEFINED_VARIABLE"
```

Treats references to unset variables as errors. For optional variables, use defaults such as:

```bash
echo "${VALUE:-default}"
```

### 4. `set -o pipefail`

```bash
set -o pipefail
false | cat
```

Makes a pipeline report failure when a command within the pipeline fails, rather than considering only the last command's status.

### 5. `set -x`

```bash
set -x
name="DevOps"
echo "$name"
set +x
```

Prints commands as they execute. Avoid tracing passwords, tokens, and other secrets.

### 6. Trap and Cleanup

```bash
TEMP_FILE=""

cleanup() {
    if [ -n "$TEMP_FILE" ]; then
        rm -f -- "$TEMP_FILE"
    fi

    echo "Cleanup complete"
}

trap cleanup EXIT
TEMP_FILE=$(mktemp)
```

The `EXIT` trap runs when the shell exits. Only remove temporary files your script created and controls.

### Recommended Safety Pattern

```bash
#!/bin/bash
set -Eeuo pipefail

if [ "$#" -lt 1 ]; then
    echo "Usage: $0 <file>" >&2
    exit 1
fi

file="$1"

if [ ! -f "$file" ]; then
    echo "Error: File not found: $file" >&2
    exit 1
fi

printf 'Processing: %s\n' "$file"
wc -l "$file"
```

Strict mode is a useful starting point, but commands that may legitimately return non-zero should still be handled deliberately.

---

## Task 8: Quick Reference Table

| Topic | Key Syntax | Example |
|---|---|---|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| If | `if [ condition ]; then` | `if [ -f "$file" ]; then` |
| For loop | `for i in list; do` | `for i in 1 2 3; do echo "$i"; done` |
| Function | `name() { ...; }` | `greet() { echo "Hi"; }` |
| Grep | `grep pattern file` | `grep -i "error" app.log` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed 's/foo/bar/g' config.txt` |
| File test | `[ -f "$file" ]` | Check a regular file |
| Exit code | `$?` | `status=$?` |
| Strict mode | `set -Eeuo pipefail` | Enable common safety checks |

---

## Commands and Tools Summary

| Tool | Primary Use |
|---|---|
| `grep` | Search text by pattern |
| `awk` | Process fields and records |
| `sed` | Transform or filter text |
| `cut` | Extract simple delimited fields |
| `sort` | Sort lines |
| `uniq` | Identify adjacent duplicate lines |
| `tr` | Translate or delete characters |
| `wc` | Count lines, words, bytes, and characters |
| `head` / `tail` | Inspect file beginnings and endings |
| `find` | Locate files by name, type, or age |
| `systemctl` | Inspect or manage system services |
| `df` | Check filesystem disk usage |
| `jq` | Parse JSON |
| `bash -n` | Check Bash syntax |
| `trap` | Run cleanup code on shell events |

---

## What I Learned: 3 Key Points

1. **Control flow and reusable scripts:** I can use conditionals, loops, functions, and arguments to automate repetitive Linux tasks.
2. **Text processing and troubleshooting:** I can combine commands such as `grep`, `awk`, `sed`, and `sort` to inspect logs and summarize errors.
3. **Safer automation:** I understand quoting, exit codes, strict-mode options, debugging, and cleanup traps.

---




## Final Note

This cheat sheet is a living reference. Keep adding examples from your daily DevOps practice, test commands before using them on production systems, and focus on understanding what each command does rather than memorizing syntax.
