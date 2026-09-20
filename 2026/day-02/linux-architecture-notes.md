# Day-02 (Linux Under the Hood)

## 1. Core Components

* **Kernel**

  * Core of Linux(heart of linux).
  * Manages CPU, RAM, storage, devices, networking, and processes.
  * Provides a bridge between applications and hardware.
  * Where linus torvalds store the code of linux.

* **User Space**

  * Where normal applications and commands run.
  * Examples: Bash, Python, `ls`, `vim`, browsers.
  * Applications interact with the kernel through **system calls(Shell Commands)**.

* **Init / systemd**

  * The first major process started by the kernel, usually **PID 1**.
  * `systemd` manages services, boot process, processes, logs, and system targets.
  * Example: starting SSH automatically when the system boots.

## 2. How Processes Work

* A **process** is a running instance of a program.
* Every process has a unique **PID (Process ID)**.
* A process can create another process called a **child process**.
* Linux commonly creates processes using:

  * `fork()` → creates a new process.
  * `exec()` → replaces the process with a new program.
* The kernel schedules processes so they can share CPU time.

## 3. What systemd Does

* Starts and stops services.
* Manages services during boot.
* Restarts failed services when configured.
* Manages dependencies between services.
* Collects system logs through **journald**.
* Useful commands:

  * `systemctl status ssh`
  * `systemctl start ssh`
  * `systemctl stop ssh`

## 4. Process States

* **Running (R)** → currently running or ready to run.
* **Sleeping (S)** → waiting for an event or resource.
* **Disk Sleep (D)** → waiting for I/O and usually cannot be interrupted immediately.
* **Stopped (T)** → execution has been paused.
* **Zombie (Z)** → process has finished, but its parent has not collected its exit status.

Check processes with:
`ps aux`

## 5. 5 Daily Linux Commands

1. `pwd` → shows current directory.
2. `ls` → lists files and directories.
3. `cd` → changes directory.
4. `ps` → views running processes.
5. `systemctl` → manages systemd services.

### Simple Flow

### How Linux Works
**Hardware → Kernel → User Space(Shell) → Applications**

### How Systemd Works
**Kernel → PID 1 (systemd) → Services → Processes**

## Screenshots 
![Linux Commands](./screenshots/5linux-command.png)
![Day 2](./screenshots/Day2.png)


# Architecture of Linux

A = Application

S = Shell

K = kernel 

# Linux Commands for daily uses 

pwd      # show current directory

ls       # list files and folders

cd       # change directory

mkdir    # create new folder

touch    # create empty file

vim      # open and edit files

cat      # display file content

head     # show first lines of a file

tail     # show last lines of a file

echo     # print text on terminal

man      # show command manual/help








