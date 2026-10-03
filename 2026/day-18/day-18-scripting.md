# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## 📌 Overview

Today I focused on writing cleaner, reusable, and safer Bash scripts.

### Topics Covered

* Bash Functions
* Function Arguments
* Return Values
* `set -euo pipefail`
* Undefined Variables
* Command Failure Handling
* Pipeline Failure Handling
* Local Variables
* System Information
* CPU, Memory, Disk Monitoring
* Modular Script Design

---

# 🎯 Task 1: Basic Functions

## Objective

Create reusable functions and pass arguments to them.

### Requirements

Create `functions.sh` with:

* A `greet` function that accepts a name
* An `add` function that accepts two numbers
* Call both functions from the script

## Example

```bash
#!/bin/bash

greet() {
    echo "Hello, $1!"
}

add() {
    echo "Sum: $(($1 + $2))"
}

greet "Devesh"
add 10 20
```

## Run

```bash
chmod +x functions.sh
./functions.sh
```

## Expected Output

```text
Hello, Devesh!
Sum: 30
```

## 📸 Screenshot

> Add a screenshot of `functions.sh` execution here.

`![Task 1 - Basic Functions](screenshots/day18-task1-functions.png)`

## What I Learned

1. Functions make Bash scripts reusable and organized.
2. Function arguments can be accessed using `$1`, `$2`, etc.
3. The same function can be called multiple times with different arguments.

---

# 🎯 Task 2: Functions with Return Values

## Objective

Use functions to collect and display system information.

### Example

```bash
#!/bin/bash

check_disk() {
    df -h /
}

check_memory() {
    free -h
}

echo "===== Disk Usage ====="
check_disk

echo ""
echo "===== Memory Usage ====="
check_memory
```

## Run

```bash
chmod +x disk_check.sh
./disk_check.sh
```

## Expected Output

```text
===== Disk Usage =====
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        20G  8.5G   12G  42% /

===== Memory Usage =====
               total        used        free
Mem:           7.7Gi       1.2Gi       4.5Gi
Swap:             0B          0B          0B
```

> Output varies depending on the system.

## 📸 Screenshot

`![Task 2 - Disk and Memory](screenshots/day18-task2-disk-memory.png)`

## What I Learned

1. Functions can execute commands and produce output.
2. `df -h /` checks disk usage of the root filesystem.
3. `free -h` displays memory usage in a human-readable format.

---

# 🎯 Task 3: Strict Mode — `set -euo pipefail`

## Objective

Learn how Bash strict mode makes scripts safer and helps detect errors early.

Use:

```bash
set -euo pipefail
```

---

## Explanation of `set -euo pipefail`

`set -euo pipefail` combines three Bash options:

### `set -e`

```bash
set -e
```

Means:

> Exit the script when a command returns a non-zero exit status.

Example:

```bash
#!/bin/bash

set -e

echo "Before failure"

ls /does-not-exist

echo "This line will not execute"
```

Because `ls` fails, the script stops immediately.

---

### `set -u`

```bash
set -u
```

Means:

> Treat an unset or undefined variable as an error.

Example:

```bash
#!/bin/bash

set -u

echo "$undefined_variable"

echo "This line will not execute"
```

Expected error:

```text
undefined_variable: unbound variable
```

This helps catch spelling mistakes and accidentally missing variables.

---

### `set -o pipefail`

```bash
set -o pipefail
```

Normally, a pipeline can hide an earlier command's failure.

For example:

```bash
false | echo "Hello"
```

With `pipefail` enabled, Bash considers the pipeline failed if any command in the pipeline fails.

Example:

```bash
#!/bin/bash

set -o pipefail

if false | grep "hello"; then
    echo "Pipeline succeeded"
else
    echo "Pipeline failed"
fi
```

Expected output:

```text
Pipeline failed
```

---

## Combined Strict Mode

Instead of writing:

```bash
set -e
set -u
set -o pipefail
```

we can write:

```bash
set -euo pipefail
```

### In simple terms

```text
-e  → Stop when a command fails
-u  → Catch undefined variables
pipefail → Detect failures inside pipelines
```

This is a common pattern for writing safer Bash scripts.

## 📸 Screenshot

> Add a screenshot showing the strict mode tests and terminal output.

`![Task 3 - Strict Mode](screenshots/day18-task3-strict-mode.png)`

## What I Learned

1. `set -e` makes scripts fail fast instead of continuing after unexpected command failures.
2. `set -u` catches undefined variables and helps prevent variable-related bugs.
3. `set -o pipefail` makes pipeline failures visible instead of allowing them to be hidden.

---

# 🎯 Task 4: Local Variables

## Objective

Understand the difference between `local` variables and regular variables inside functions.

## Function Using `local`

```bash
local_demo() {
    local name="Devesh"
    local role="DevOps Learner"

    echo "Inside local_demo:"
    echo "Name: $name"
    echo "Role: $role"
}

local_demo

echo ""
echo "Outside local_demo:"
echo "Name: ${name:-Not available}"
echo "Role: ${role:-Not available}"
```

### Expected Output

```text
Inside local_demo:
Name: Devesh
Role: DevOps Learner

Outside local_demo:
Name: Not available
Role: Not available
```

## Function Using Regular Variables

```bash
regular_demo() {
    name="Manhor Kumar Das"
    role="AWS Engineer"

    echo "Inside regular_demo:"
    echo "Name: $name"
    echo "Role: $role"
}

regular_demo

echo ""
echo "Outside regular_demo:"
echo "Name: $name"
echo "Role: $role"
```

### Expected Output

```text
Inside regular_demo:
Name: Manhor Kumar Das
Role: AWS Engineer

Outside regular_demo:
Name: Manhor Kumar Das
Role: AWS Engineer
```

## Comparison

| Variable           | Inside Function | Outside Function |
| ------------------ | --------------: | ---------------: |
| `local name="..."` |               ✅ |                ❌ |
| `name="..."`       |               ✅ |                ✅ |

## 📸 Screenshot

`![Task 4 - Local Variables](screenshots/day18-task4-local-variables.png)`

## What I Learned

1. `local` keeps a variable limited to the current function.
2. Regular variables can remain available after a function finishes.
3. Using `local` helps prevent accidental changes to variables outside the function.

---

# 🚀 Task 5: Build a Script — System Info Reporter

## Objective

Build an intermediate Bash script using functions and strict mode.

### Requirements

1. Print hostname and OS information.
2. Print system uptime.
3. Print top 5 disk usage.
4. Print memory usage.
5. Print top 5 CPU-consuming processes.
6. Use a `main` function.
7. Use section headers.
8. Use `set -euo pipefail`.

## Complete Script

```bash
#!/bin/bash

set -euo pipefail

print_system_info() {
    echo "Hostname: $(hostname)"
    echo "OS: $(grep '^PRETTY_NAME=' /etc/os-release | cut -d= -f2- | tr -d '"')"
}

print_uptime() {
    uptime
}

print_disk_usage() {
    df -h --output=source,size,used,avail,pcent,target \
        | sort -k5 -hr \
        | head -n 6
}

print_memory_usage() {
    free -h
}

print_cpu_processes() {
    ps aux --sort=-%cpu | head -n 6
}

main() {

    echo "========================================"
    echo "        SYSTEM INFORMATION"
    echo "========================================"

    echo ""
    echo "----- Hostname & OS -----"
    print_system_info

    echo ""
    echo "----- Uptime -----"
    print_uptime

    echo ""
    echo "----- Top 5 Disk Usage -----"
    print_disk_usage

    echo ""
    echo "----- Memory Usage -----"
    print_memory_usage

    echo ""
    echo "----- Top 5 CPU Processes -----"
    print_cpu_processes
}

main
```

## Run

```bash
chmod +x system_info.sh
./system_info.sh
```

## Example Output

```text
========================================
        SYSTEM INFORMATION
========================================

----- Hostname & OS -----
Hostname: ip-172-31-43-201
OS: Ubuntu 24.04 LTS

----- Uptime -----
17:30:21 up 2 days, 4:21, 1 user, load average: 0.08, 0.04, 0.01

----- Top 5 Disk Usage -----
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        20G  8.5G   12G  42% /

----- Memory Usage -----
               total        used        free
Mem:           7.7Gi       1.2Gi       4.5Gi
Swap:             0B          0B          0B

----- Top 5 CPU Processes -----
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.5  ...
```

> System values will vary depending on the machine.

## 📸 Screenshot

`![Task 5 - System Info Reporter](screenshots/day18-task5-system-info.png)`

---

# 🧠 Day 18 – What I Learned

### 3 Key Points

1. **Functions make Bash scripts cleaner and reusable**
   I learned how to divide a large script into smaller functions, pass arguments, and use a `main` function to control execution.

2. **Strict mode makes scripts safer**
   I practiced `set -euo pipefail` to catch command failures, undefined variables, and pipeline failures early.

3. **Local variables improve script reliability**
   I learned how `local` prevents function variables from leaking into the rest of the script and helps avoid unexpected variable conflicts.

---

🚀 **Day 18 Complete!**
