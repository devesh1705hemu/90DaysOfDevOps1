# Linux File System Hierarchy & Real-World Troubleshooting

## 1. Linux File System Hierarchy

Linux uses a hierarchical file system that starts from the root directory `/`.

### Important Directories

| Directory | Purpose |
|---|---|
| `/` | Root directory |
| `/bin` | Essential user commands |
| `/sbin` | System administration commands |
| `/etc` | Configuration files |
| `/home` | User home directories |
| `/root` | Root user's home directory |
| `/var` | Variable data |
| `/var/log` | System and application logs |
| `/tmp` | Temporary files |
| `/usr` | User programs and libraries |
| `/opt` | Optional/third-party software |
| `/dev` | Device files |
| `/proc` | Process and kernel information |
| `/sys` | Hardware and kernel information |
| `/run` | Runtime system data |
| `/mnt` | Temporary mount points |
| `/media` | Removable media |
| `/boot` | Bootloader and kernel files |

---

# 2. Important Directories for DevOps

```text
/etc       → Configuration
/var/log   → Logs
/home      → User files
/opt       → Applications
/tmp       → Temporary files
/proc      → Process information
/dev       → Devices
/run       → Runtime information
/boot      → Boot files
/usr       → Programs and libraries


```
#  Real-World Troubleshooting Scenarios
##  Scenario 1: Service Not Starting

### Problem
A web application service called myapp failed to start after a server reboot.

#### Steps

Step 1: Check service status

systemctl status myapp

Step 2: Check service logs

journalctl -u myapp -n 50

Step 3: Check whether the service is enabled

systemctl is-enabled myapp

Step 4: Check logs from the current boot

journalctl -u myapp -b

Step 5: Check service configuration

systemctl cat myapp

Step 6: Restart the service after fixing the issue

sudo systemctl restart myapp

Step 7: Verify the service

systemctl status myapp

## Scenario 2: High CPU Usage
### Problem
The application server is slow and you need to identify the process using high CPU.

#### Steps

Step 1: Check live CPU usage

top

Step 2: Find processes sorted by CPU usage

ps aux --sort=-%cpu | head -10

Step 3: Note the PID of the process using high CPU

PID

Step 4: Check details of the process

ps -p <PID> -f

Step 5: Inspect the process

ls -l /proc/<PID>

Step 6: Monitor the specific process

top -p <PID>

## Scenario 3: Finding Service Logs
### Problem
A developer asks where the logs for the docker service are located.

#### Steps

Step 1: Check Docker service status

systemctl status docker

Step 2: View the last 50 Docker logs

journalctl -u docker -n 50

Step 3: View Docker logs from the current boot

journalctl -u docker -b

Step 4: Follow Docker logs in real time

journalctl -u docker -f

Step 5: View logs from the last hour

journalctl -u docker --since "1 hour ago"

## Scenario 4: File Permissions Issue
### Problem
A script at /home/user/backup.sh gives a Permission denied error when executed.

#### Steps

Step 1: Check current permissions

ls -l /home/user/backup.sh

Step 2: Check whether execute permission is missing

Check for x permission

Step 3: Add execute permission

chmod +x /home/user/backup.sh

Step 4: Verify permissions

ls -l /home/user/backup.sh



# 🛠️ Linux Troubleshooting Common Commands with Uses

## System

| Command | Use |
|---|---|
| `uptime` | Check how long the system has been running and view system load |
| `hostname` | Display the system's hostname |
| `uname -a` | Display kernel and system information |
| `whoami` | Show the currently logged-in user |

---

## CPU

| Command | Use |
|---|---|
| `top` | Monitor CPU usage and running processes in real time |
| `ps aux --sort=-%cpu \| head -10` | Find the top 10 processes consuming the most CPU |

---

## Memory

| Command | Use |
|---|---|
| `free -h` | Check RAM and swap memory usage |
| `ps aux --sort=-%mem \| head -10` | Find the top 10 processes consuming the most memory |

---

## Disk

| Command | Use |
|---|---|
| `df -h` | Check available and used disk space |
| `du -sh *` | Check the size of files and directories in the current directory |
| `df -i` | Check inode usage and availability |

---

## Processes

| Command | Use |
|---|---|
| `ps aux` | List all running processes |
| `ps -ef` | Display detailed information about running processes |
| `pgrep <process>` | Find the PID of a process by its name |
| `ps -p <PID> -f` | Display detailed information about a specific process |

---

## Services

| Command | Use |
|---|---|
| `systemctl status <service>` | Check the current status of a service |
| `systemctl restart <service>` | Restart a service |
| `systemctl is-enabled <service>` | Check whether a service is configured to start automatically at boot |

---

## Logs

| Command | Use |
|---|---|
| `journalctl -u <service> -n 50` | View the last 50 log entries for a service |
| `journalctl -u <service> -f` | Follow service logs in real time |
| `journalctl -p err` | View error-level system logs |

---

## Network

| Command | Use |
|---|---|
| `ip addr` | Display network interfaces and IP addresses |
| `ip route` | Display the system's routing table |
| `ping <host>` | Test network connectivity to a host |
| `curl <URL>` | Test HTTP/HTTPS connectivity and application responses |
| `ss -tulnp` | Display listening TCP/UDP ports and associated processes |
| `dig <domain>` | Query DNS information for a domain |

---

## Files

| Command | Use |
|---|---|
| `ls -la` | List all files, including hidden files, with detailed information |
| `find <path> -name <file>` | Search for a file by name |
| `stat <file>` | Display detailed file information such as permissions, owner, size, and timestamps |
| `file <file>` | Identify the type of a file |

---

## Permissions

| Command | Use |
|---|---|
| `ls -l <file>` | Check file permissions, owner, and group |
| `chmod +x <file>` | Add execute permission to a file |
| `chown <user>:<group> <file>` | Change the owner and group of a file |

---

# 🔥 Quick Troubleshooting Flow

## Server Slow

```bash
top
ps aux --sort=-%cpu | head -10
free -h
ps aux --sort=-%mem | head -10

Step 5: Execute the script

./backup.sh

Alternative: Execute using Bash

bash /home/user/backup.sh
