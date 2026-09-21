# Linux File Read/Write Practice

## Objective

Practice basic Linux file read/write operations using fundamental commands.

### Goals

* Create a text file
* Write text to a file using redirection
* Append new lines to an existing file
* Read the complete file using `cat`
* Read specific parts using `head` and `tail`
* Use `tee` to write and display content simultaneously

---

# 1. Create a File

Create an empty file named `notes.txt`.

```bash
touch notes.txt
```

### Use of `touch`

The `touch` command is used to:

* Create a new empty file
* Update the access and modification timestamp of an existing file

### Verify the File

```bash
ls
```

Expected output:

```text
notes.txt
```

---

# 2. Write Text Using `>`

Write the first line into the file:

```bash
echo "Linux file handling practice" > notes.txt
```

### Use of `>`

The `>` redirection operator:

* Writes command output into a file
* Creates the file if it does not exist
* **Overwrites existing content** in the file

---

# 3. Append Text Using `>>`

Add the second line:

```bash
echo "Learning basic file read and write commands" >> notes.txt
```

Add the third line:

```bash
echo "Practicing Linux commands every day" >> notes.txt
```

### Use of `>>`

The `>>` redirection operator:

* Adds output to the end of a file
* Creates the file if it does not exist
* **Does not overwrite existing content**

---

# 4. Read the Complete File Using `cat`

Display the entire file:

```bash
cat notes.txt
```

Expected output:

```text
Linux file handling practice
Learning basic file read and write commands
Practicing Linux commands every day
```

### Use of `cat`

`cat` is commonly used to:

* Display the contents of a file
* Read a complete text file
* Combine multiple files
* Create files using standard input

---

# 5. Read the Beginning Using `head`

Display the first lines of the file:

```bash
head notes.txt
```

By default, `head` displays the first **10 lines**.

You can specify the number of lines:

```bash
head -n 2 notes.txt
```

Expected output:

```text
Linux file handling practice
Learning basic file read and write commands
```

### Use of `head`

`head` is used to:

* View the beginning of a file
* Quickly inspect log files
* Display a specific number of starting lines

---

# 6. Read the End Using `tail`

Display the last lines of the file:

```bash
tail notes.txt
```

By default, `tail` displays the last **10 lines**.

Display only the last 2 lines:

```bash
tail -n 2 notes.txt
```

Expected output:

```text
Learning basic file read and write commands
Practicing Linux commands every day
```

### Use of `tail`

`tail` is used to:

* View the end of a file
* Check recent entries in log files
* Display a specific number of ending lines

---

# 7. Use `tee` to Write and Display

Use `tee` to write content to the file while displaying it on the terminal:

```bash
echo "Linux practice with tee" | tee tee-test.txt
```

Expected terminal output:

```text
Linux practice with tee
```

Check the file:

```bash
cat tee-test.txt
```

Expected output:

```text
Linux practice with tee
```

### Use of `tee`

The `tee` command is used to:

* Display command output on the terminal
* Write the same output into a file
* Send output to multiple destinations when combined with pipelines

### Append Using `tee`

`tee` can also append instead of overwrite:

```bash
echo "Another line" | tee -a tee-test.txt
```

Here, `-a` means **append**.

---

# 8. Command Summary

| Command / Operator | Purpose                                 |
| ------------------ | --------------------------------------- |
| `touch`            | Create an empty file                    |
| `echo`             | Print text                              |
| `>`                | Write/overwrite a file                  |
| `>>`               | Append to a file                        |
| `cat`              | Display the complete file               |
| `head`             | Display the beginning of a file         |
| `tail`             | Display the end of a file               |
| `tee`              | Write and display output simultaneously |
| `tee -a`           | Append and display output               |

---


# Practice Screenshots:

# Summary of the day:
