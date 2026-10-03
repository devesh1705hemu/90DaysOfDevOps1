# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Objective

Level up Shell Scripting by practicing:

* `for` loops
* `while` loops
* Command-line arguments
* Package installation through scripts
* Basic error handling
* Root privilege validation

---

# Task 1 – For Loop

## 1.1 `for_loop.sh`

### Objective

Create a script that loops through a list of 5 fruits and prints each fruit.

### Script

```bash
#!/bin/bash

fruits=("Apple" "Banana" "Mango" "Orange" "Grapes")

for fruit in "${fruits[@]}"
do
    echo "Fruit: $fruit"
done
```

### Output

Add a screenshot of your terminal output here.

```text
Example:

Fruit: Apple
Fruit: Banana
Fruit: Mango
Fruit: Orange
Fruit: Grapes
```

![For Loop Output](./screenshots/for-loop-output.png)

### What I Learned

* I learned how to create and use a `for` loop.
* I practiced iterating through a list of values.
* I learned how loops can reduce repetitive commands.

---

## 1.2 `count.sh`

### Objective

Print numbers from 1 to 10 using a `for` loop.

### Script

```bash
#!/bin/bash

for i in {1..10}
do
    echo "Number: $i"
done
```

### Output

Add a screenshot of your terminal output here.

```text
Example:

Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
Number: 6
Number: 7
Number: 8
Number: 9
Number: 10
```

![Count Output](./screenshots/count-output.png)

### What I Learned

* I learned how to use numeric ranges in a `for` loop.
* I practiced controlling the starting and ending values.
* I understood how loops automate repetitive operations.

---

# Task 2 – While Loop

## `countdown.sh`

### Objective

Create a script that:

* Takes a number from the user.
* Counts down to `0`.
* Uses a `while` loop.
* Prints `Done!` after the countdown.

### Script

```bash
#!/bin/bash

read -p "Enter a number: " number

while [ "$number" -ge 0 ]
do
    echo "$number"
    number=$((number - 1))
done

echo "Done!"
```

### Output

Add a screenshot of your terminal output here.

```text
Example:

Enter a number: 5
5
4
3
2
1
0
Done!
```

![Countdown Output](./screenshots/countdown-output.png)

### What I Learned

* I learned how to use a `while` loop.
* I practiced taking input from the user.
* I learned how to update a variable inside a loop.
* I understood how a loop condition controls execution.

---

# Task 3 – Command-Line Arguments

## 3.1 `greet.sh`

### Objective

Create a script that:

* Accepts a name using `$1`.
* Prints `Hello, <name>!`.
* Displays usage information if no argument is provided.

### Script

```bash
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Usage: ./greet.sh <name>"
    exit 1
fi

echo "Hello, $1!"
```

### Output With Argument

Example:

```bash
./greet.sh Devesh
```

```text
Hello, Devesh!
```

![Greet Output](./screenshots/greet-output.png)

### Output Without Argument

Example:

```bash
./greet.sh
```

```text
Usage: ./greet.sh <name>
```

### What I Learned

* `$1` represents the first command-line argument.
* `$#` represents the total number of arguments.
* Arguments allow scripts to receive dynamic input.
* Input validation helps prevent incorrect script execution.

---

## 3.2 `args_demo.sh`

### Objective

Demonstrate:

* `$0` → Script name
* `$#` → Number of arguments
* `$@` → All arguments

### Script

```bash
#!/bin/bash

echo "Script name: $0"
echo "Total arguments: $#"
echo "All arguments: $@"
```

### Example Command

```bash
./args_demo.sh Linux Docker AWS
```

### Output

```text
Script name: ./args_demo.sh
Total arguments: 3
All arguments: Linux Docker AWS
```

![Arguments Demo Output](./screenshots/args-demo-output.png)

### What I Learned

* `$0` stores the script name.
* `$#` gives the total number of arguments.
* `$@` represents all supplied arguments.
* Command-line arguments make scripts more reusable.

---

# Task 4 – Install Packages via Script

## `install_packages.sh`

### Objective

Create a script that:

* Defines a list of packages:

  * `nginx`
  * `curl`
  * `wget`
* Loops through the package list.
* Checks whether each package is installed.
* Installs missing packages.
* Skips packages that are already installed.
* Displays the status of each package.
* Checks whether the script is being run as root.

### Prerequisite

Run the script with root privileges:

```bash
sudo -i
```

or:

```bash
sudo su
```

You can also run the script directly with:

```bash
sudo ./install_packages.sh
```

### Script

```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]
then
    echo "Error: Please run this script as root."
    echo "Use: sudo ./install_packages.sh"
    exit 1
fi

packages=("nginx" "curl" "wget")

echo "Checking required packages..."

for package in "${packages[@]}"
do
    if dpkg -s "$package" &> /dev/null
    then
        echo "$package is already installed."
    else
        echo "$package is not installed. Installing..."

        apt update -y
        apt install -y "$package"

        if [ $? -eq 0 ]
        then
            echo "$package installed successfully."
        else
            echo "Failed to install $package."
        fi
    fi
done

echo "Package check completed."
```

### Output

Add a screenshot of your actual terminal output here.

Example when packages are already installed:

```text
Checking required packages...
nginx is already installed.
curl is already installed.
wget is already installed.
Package check completed.
```

Example when a package is missing:

```text
Checking required packages...
nginx is already installed.
curl is already installed.
wget is not installed. Installing...
wget installed successfully.
Package check completed.
```

![Install Packages Output](./screenshots/install-packages-output.png)

### What I Learned

* I learned how to check whether a package is installed.
* I practiced installing multiple packages using a loop.
* I learned why administrative scripts require root privileges.
* I practiced checking command success using `$?`.

---

# Task 5 – Error Handling

## 5.1 `safe_script.sh`

### Objective

Create a script that:

* Uses `set -e`.
* Creates `/tmp/devops-test`.
* Navigates into the directory.
* Creates a file inside the directory.
* Uses `||` for basic error handling.

### Script

```bash
#!/bin/bash

set -e

echo "Starting safe script..."

mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test || {
    echo "Error: Failed to enter directory"
    exit 1
}

touch test.txt || {
    echo "Error: Failed to create file"
    exit 1
}

echo "File created successfully."
echo "Safe script completed."
```

### Output

Add a screenshot of your actual terminal output here.

Example:

```text
Starting safe script...
Directory already exists
File created successfully.
Safe script completed.
```

![Safe Script Output](./screenshots/safe-script-output.png)

### What I Learned

* `set -e` makes the script exit when an unhandled command fails.
* `||` can be used to execute an alternative command when a command fails.
* Error handling makes scripts safer and easier to debug.
* `exit` can be used to terminate a script with an error status.

---

## 5.2 Root Privilege Validation

The `install_packages.sh` script checks whether it is being executed as root.

### Root Check

```bash
if [ "$EUID" -ne 0 ]
then
    echo "Error: Please run this script as root."
    echo "Use: sudo ./install_packages.sh"
    exit 1
fi
```

### Output Without Root

```text
Error: Please run this script as root.
Use: sudo ./install_packages.sh
```

### Output With Root

```text
Checking required packages...
...
Package check completed.
```

### What I Learned

* `$EUID` can be used to check the effective user ID.
* Root validation prevents permission-related failures.
* `exit 1` indicates that the script ended because of an error.
* Administrative scripts should validate permissions before performing system-level operations.

---

# Shell Scripting Concepts Covered

| Concept       | Purpose                                   |   |                                                   |
| ------------- | ----------------------------------------- | - | ------------------------------------------------- |
| `for`         | Repeat commands for a list or range       |   |                                                   |
| `while`       | Repeat commands while a condition is true |   |                                                   |
| `$0`          | Script name                               |   |                                                   |
| `$1`          | First command-line argument               |   |                                                   |
| `$2`          | Second command-line argument              |   |                                                   |
| `$#`          | Number of arguments                       |   |                                                   |
| `$@`          | All command-line arguments                |   |                                                   |
| `$?`          | Exit status of the previous command       |   |                                                   |
| `$EUID`       | Effective user ID                         |   |                                                   |
| `set -e`      | Exit when an unhandled command fails      |   |                                                   |
| `             |                                           | ` | Execute a command when the previous command fails |
| `exit`        | Exit the script                           |   |                                                   |
| `dpkg -s`     | Check Debian/Ubuntu package status        |   |                                                   |
| `apt install` | Install packages                          |   |                                                   |

---


# Key Learnings

Today I practiced writing more practical and automation-oriented Shell Scripts.

### 1. Loops

I practiced `for` and `while` loops to automate repetitive operations.

### 2. Command-Line Arguments

I learned how `$0`, `$1`, `$#`, and `$@` allow scripts to receive and process external input.

### 3. Package Automation

I created a script that checks and installs required packages automatically.

### 4. Error Handling

I practiced using `set -e`, `||`, `$?`, and `exit` to handle script failures.

### 5. Permission Management

I learned how to check whether a script is running with root privileges before performing administrative operations.


### What I Learned

* 🔄 Learned to use **`for` and `while` loops** to automate repetitive tasks in Bash.
* 🧩 Practiced **command-line arguments** using `$1`, `$#`, `$@`, and `$0` to make scripts more flexible.
* 🛡️ Learned **basic error handling and automation**, including `set -e`, `||`, root privilege checks, and package installation scripts.


---


