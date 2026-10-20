# Day 28 – Revision Day: Everything from Day 1 to Day 27

## Objective

Revise the concepts learned during Days 1–27 of the #90DaysOfDevOps challenge. Strengthen my knowledge of Linux, networking, shell scripting, storage management, Git, and GitHub while identifying areas that require additional practice.

## Task 1: Self-Assessment Checklist

**Assessment legend:**
- [x] Can do confidently
- [ ] Need to revisit
- [ ] Haven't done yet

Note: These are revision targets, not verified claims about practical proficiency. Update the checkboxes according to actual hands-on performance.

### Linux

- [x] Navigate the file system and create, move, and delete files and directories.
- [x] Manage processes using `ps`, `top`, `kill`, and job control.
- [x] Manage services using `systemctl`.
- [x] Read and edit text files using `vi`, `vim`, or `nano`.
- [x] Troubleshoot CPU, memory, and disk usage using `top`, `free`, `df`, and `du`.
- [x] Explain the Linux file system hierarchy, including `/`, `/etc`, `/var`, `/home`, and `/tmp`.
- [x] Create users and groups and manage passwords.
- [x] Set file permissions using `chmod` in numeric and symbolic modes.
- [x] Change file ownership using `chown` and `chgrp`.
- [ ] Create and manage LVM volumes independently.
- [x] Check network connectivity using `ping`, `curl`, `ss`, `dig`, and `nslookup`.
- [x] Explain DNS resolution, IP addressing, subnets, and common ports.

### Shell Scripting

- [x] Write scripts using variables, arguments, and user input.
- [x] Use `if`, `elif`, `else`, and `case` statements.
- [x] Write `for`, `while`, and `until` loops.
- [x] Define functions and pass arguments.
- [x] Process text using `grep`, `awk`, `sed`, `sort`, and `uniq`.
- [ ] Handle errors reliably using `set -e`, `set -u`, `set -o pipefail`, and `trap`.
- [x] Schedule scripts using `crontab`.

### Git & GitHub

- [x] Initialize a repository, stage changes, commit, and view history.
- [x] Create and switch branches.
- [x] Push and pull changes from GitHub.
- [x] Explain cloning versus forking.
- [x] Understand fast-forward merges and merge commits.
- [x] Rebase branches and compare rebase with merge.
- [x] Use `git stash` and `git stash pop`.
- [x] Cherry-pick a commit from another branch.
- [x] Explain squash merges versus regular merges.
- [] Understand `git reset` and `git revert`.
- [] Explain GitFlow, GitHub Flow, and Trunk-Based Development.
- [ ] Practise creating repositories, issues, and pull requests using GitHub CLI.

**Areas selected for further revision:**
1. LVM volume management.
2. Shell scripting error handling.
3. GitHub CLI workflows.

## Task 2: Revisit Your Weak Spots

### Topic 1: LVM Volume Management

**What is LVM?**

Logical Volume Manager provides flexible storage management in Linux. It allows physical storage devices to be combined into volume groups and divided into logical volumes.

**Important components:**
- Physical Volume (PV): A disk or partition prepared for LVM.
- Volume Group (VG): A storage pool made from one or more physical volumes.
- Logical Volume (LV): A virtual block device allocated from a volume group.

**Commands revised:**

```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
df -h
sudo pvcreate /dev/sdb
sudo vgcreate vg_data /dev/sdb
sudo lvcreate -L 2G -n lv_data vg_data
```

**Practical note:** The final three commands modify storage and must only be run on a suitable, verified, unused disk in a test environment. They are examples, not commands to run blindly on a production machine.

**What I learned:** LVM makes storage allocation and supported resizing more flexible than fixed partitions. Before making storage changes, I must verify the target device, available space, filesystem, and backup status.

### Topic 2: Shell Scripting Error Handling

Error handling makes scripts more reliable and easier to troubleshoot.

**Important options:**
- `set -e`: Exits on many unhandled command failures.
- `set -u`: Treats references to unset variables as errors.
- `set -o pipefail`: Reports a pipeline failure if a command within the pipeline fails.
- `trap`: Executes a handler when a specified signal or shell condition occurs.

**Practice script:**

```bash
#!/usr/bin/env bash
set -euo pipefail

cleanup() {
    echo "Script execution finished."
}

trap cleanup EXIT

if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

target_dir="$1"

if [[ ! -d "$target_dir" ]]; then
    echo "Error: Directory does not exist."
    exit 1
fi

find "$target_dir" -maxdepth 1 -type f -print
echo "Directory inspection completed."
```

**What I learned:** Validating inputs, checking exit statuses, and using appropriate error handling prevents scripts from silently continuing after many failures. These options have exceptions, so critical operations still need explicit checks.

### Topic 3: GitHub CLI

GitHub CLI allows developers to manage GitHub resources from the terminal.

**Useful commands:**

```bash
gh auth status
gh repo list
gh repo create
gh issue list
gh issue create
gh pr list
gh pr create
gh pr view
```

**Typical workflow:**

1. Authenticate using `gh auth login`.
2. List repositories using `gh repo list`.
3. Create or clone a repository.
4. Create a feature branch and commit changes.
5. Push the branch to GitHub.
6. Open a pull request using `gh pr create`.
7. Review pull request status using `gh pr view`.

**What I learned:** GitHub CLI reduces context switching between the terminal and browser. Authentication and repository permissions are required for operations that modify remote resources.

## Task 3: Quick-Fire Questions and Answers

### Q1. What does `chmod 755 script.sh` do?

It assigns these permissions:

- Owner: `7` = read, write, execute (`rwx`).
- Group: `5` = read and execute (`r-x`).
- Others: `5` = read and execute (`r-x`).

The file owner can modify and execute the script, while group members and other users can read and execute it, subject to directory permissions and other security controls.

### Q2. What is the difference between a process and a service?

A process is a running instance of a program with a process ID (PID). A service is a background component that provides functionality and is often managed by a service manager such as systemd. A service can involve one or more processes.

**Example:** Nginx is a web server service that runs processes to handle HTTP requests.

### Q3. How do you find which process is using port 8080?

Use:

```bash
sudo ss -ltnp 'sport = :8080'
```

Alternatively:

```bash
sudo lsof -i :8080
```

These commands help identify processes associated with port 8080. The `ss` command above checks listening TCP sockets.

### Q4. What does `set -euo pipefail` do in a shell script?

- `-e`: Exits when certain unhandled command failures occur.
- `-u`: Treats unset variables as errors.
- `-o pipefail`: Makes a pipeline fail if a command within it fails.
- Together, these options improve error detection and help prevent scripts from silently ignoring failures.

They do not replace explicit error handling because their behaviour depends on context.

### Q5. What is the difference between `git reset --hard` and `git revert`?

`git reset --hard` moves the current branch reference to the specified commit and resets the index and working tree. It can discard uncommitted changes and may rewrite local history.

`git revert` creates a new commit that reverses the changes introduced by an earlier commit.

**Recommendation:** Prefer `git revert` when undoing changes in shared branch history. Use `git reset --hard` carefully, especially when valuable uncommitted work exists.

### Q6. What branching strategy would you recommend for a team of five developers shipping weekly?

I recommend **GitHub Flow** because it is simple and suitable for a small team releasing frequently.

The workflow includes:
1. Keeping the main branch stable.
2. Creating short-lived feature branches.
3. Opening pull requests.
4. Reviewing code and running automated tests.
5. Merging approved changes into the main branch.
6. Deploying regularly through a tested release process.

### Q7. What does `git stash` do, and when would you use it?

`git stash` temporarily stores supported uncommitted changes so I can switch branches or work on another task without creating an incomplete commit.

Useful commands:

```bash
git stash
git stash list
git stash pop
git stash apply
```

`git stash pop` reapplies a stash and removes it from the stash list if the application succeeds. `git stash apply` reapplies the changes without removing the stash entry.

### Q8. How do you schedule a script to run every day at 3 AM?

Open the user's crontab:

```bash
crontab -e
```

Add the following entry, replacing the path with the actual absolute path:

```cron
0 3 * * * /bin/bash /path/to/script.sh
```

The five cron fields represent minute, hour, day of month, month, and day of week.

This runs daily at 3:00 AM according to the system's local time, assuming the cron service is running and the script has the required permissions.

### Q9. What is the difference between `git fetch` and `git pull`?

`git fetch` downloads updates from a remote repository and updates remote-tracking references without integrating the changes into the current branch.

`git pull` fetches updates and then integrates them into the current branch, typically using merge or rebase depending on configuration and options.

**Example:**

```bash
git fetch origin
git pull origin main
```

Fetching first is useful when I want to inspect remote changes before integrating them.

### Q10. What is LVM, and why would you use it instead of regular partitions?

LVM stands for Logical Volume Manager. It creates a flexible storage layer between physical storage devices and filesystems.

Its advantages include:
- Combining physical storage devices into a storage pool.
- Allocating and resizing logical volumes in supported configurations.
- Managing storage more flexibly as requirements change.
- Supporting snapshots where configured.

Unlike fixed partitions, LVM makes storage allocation more flexible, but it does not eliminate the need for backups or careful capacity planning.

## Task 4: Organize Your Work

### 1. Verify previous submissions

Check the repository status:

```bash
git status
git log --oneline -10
git remote -v
```

Inspect the daily folders:

```bash
ls 2026/
```

Confirm that Days 1–27 have been committed and pushed. Verify that the remote repository is the intended fork.

### 2. Update Git command notes

Ensure `git-commands.md` covers:

- Repository initialization and configuration.
- Staging, commits, and history.
- Branch creation and switching.
- Merge, rebase, and cherry-pick.
- Stash, reset, and revert.
- Fetch, pull, push, and remote management.
- Branching strategies and pull requests.

### 3. Update the shell scripting cheat sheet

Ensure it includes:

- Variables and positional arguments.
- Conditions, loops, and functions.
- File checks and exit statuses.
- Text processing commands.
- Error handling and traps.
- Cron scheduling.
- Practical scripts for backups and log rotation.

### 4. Verify the GitHub profile

Review the profile README, repository descriptions, pinned projects, commit history, and project documentation. Remove unnecessary files and confirm that no credentials or sensitive data are exposed.

### 5. Commit and push Day 28

From the repository root:

```bash
git add 2026/day-28/day-28-notes.md
git commit -m "docs: add day 28 revision notes"
git push origin HEAD
```

If the commit reports that there is nothing to commit, inspect `git status` before proceeding. Confirm the push succeeded and verify the file on GitHub.

## Task 5: Teach It Back

### Topic: Git Branching Explained Simply

1. Git branching allows developers to work on separate features without interfering with the main development branch.
2. The main branch contains the primary line of development.
3. A feature branch creates an independent line of work for a new feature or bug fix.
4. Developers make commits on their feature branches.
5. Once the work is ready, they open a pull request for review.
6. Teammates review the code and automated tests help verify its correctness.
7. After approval, the feature branch can be merged into the main branch.
8. This workflow makes collaboration easier and helps isolate changes.
9. Short-lived branches and frequent integration can reduce the risk of difficult merge conflicts.


## Task 6: Essential Commands and Their Uses

### 1. Linux Commands

```bash
pwd                              # Displays the current working directory.
ls -la                           # Lists all files, including hidden files, with detailed information.
mkdir test-dir                   # Creates a directory named test-dir.
touch test.txt                   # Creates an empty file or updates its timestamp.
cp test.txt test-copy.txt        # Copies a file to a new file.
mv test.txt renamed.txt          # Moves or renames a file.
rm renamed.txt                   # Deletes a file.
ps aux                           # Displays running processes and their details.
top                              # Monitors running processes and system resource usage.
jobs                             # Lists jobs running in the current shell.
bg                               # Resumes a stopped job in the background.
fg                               # Brings a background job to the foreground.
kill <PID>                       # Sends the default termination signal to a process.
free -h                          # Displays memory usage in human-readable units.
df -h                            # Displays filesystem disk space usage.
du -sh /var/log                  # Shows the total size of /var/log.
systemctl status nginx           # Displays the status of the Nginx service.
sudo systemctl start nginx       # Starts the Nginx service.
sudo systemctl stop nginx        # Stops the Nginx service.
sudo systemctl enable nginx      # Enables Nginx to start automatically at boot.
cat /etc/os-release              # Displays Linux distribution information.
nano test.txt                    # Opens test.txt in the Nano text editor.
sudo useradd testuser            # Creates a user named testuser.
sudo passwd testuser             # Sets or changes the user's password.
sudo groupadd devops              # Creates a group named devops.
sudo usermod -aG devops testuser # Adds testuser to the devops supplementary group.
chmod 755 script.sh              # Gives the owner rwx and group/others r-x permissions.
sudo chown user:group script.sh  # Changes the file owner and group.
sudo chgrp devops script.sh      # Changes the file's group ownership to devops.
lsblk                            # Lists block devices and their partitions.
sudo pvs                         # Displays LVM physical volumes.
sudo vgs                         # Displays LVM volume groups.
sudo lvs                         # Displays LVM logical volumes.
ping -c 4 google.com             # Sends four ICMP echo requests to test connectivity.
curl -I https://google.com       # Retrieves HTTP response headers from the URL.
ss -ltnp                         # Lists listening TCP sockets with numeric ports and process details.
dig google.com                   # Queries DNS records for a domain.
nslookup google.com              # Looks up DNS information for a domain.
sudo lsof -i :8080               # Lists processes using network port 8080.
```

### 2. LVM Commands

```bash
lsblk                                # Identifies disks and partitions before storage operations.
sudo pvcreate /dev/sdb               # Initializes /dev/sdb as an LVM physical volume.
sudo vgcreate vg_data /dev/sdb       # Creates the vg_data volume group from the physical volume.
sudo lvcreate -L 2G -n lv_data vg_data # Creates a 2 GB logical volume named lv_data.
```

**Warning:** Verify the target disk before running LVM commands. These operations change storage configuration and may affect existing data.

### 3. Shell Scripting Commands

```bash
bash script.sh argument1         # Runs script.sh with argument1 as its first argument.
bash -n script.sh                # Checks shell script syntax without executing it.
chmod +x script.sh               # Adds execute permission according to existing file permissions.
./script.sh                      # Executes the script from the current directory if permitted.
grep "error" app.log             # Finds lines containing the word error.
awk '{print $1}' app.log         # Prints the first whitespace-separated field from each line.
sed 's/old/new/g' file.txt       # Prints file contents with old replaced by new on each line.
sort file.txt                    # Sorts the lines in a file.
sort file.txt | uniq             # Sorts lines and removes adjacent duplicate lines.
crontab -e                       # Opens the current user's scheduled cron jobs for editing.
crontab -l                       # Lists the current user's scheduled cron jobs.
```

### 4. Shell Scripting Error Handling

```bash
set -e                 # Exits on many unhandled command failures.
set -u                 # Treats references to unset variables as errors.
set -o pipefail        # Makes a pipeline fail if any command in it fails.
trap cleanup EXIT      # Runs the cleanup function or command when the shell exits.
```

**Note:** These options improve error detection but do not replace explicit error handling. Their behaviour depends on the script's context.

### 5. Git Commands

```bash
git init                         # Initializes a new Git repository.
git status                       # Shows the working tree and staging area status.
git add .                        # Stages changes in the current directory and subdirectories.
git commit -m "Initial commit"   # Creates a commit with a descriptive message.
git log --oneline                # Displays commit history in compact format.
git log --oneline --graph --all  # Displays all branch histories as a graph.
git branch                       # Lists local branches.
git switch -c feature-branch     # Creates and switches to a new branch.
git switch main                  # Switches to the main branch.
git remote -v                    # Displays configured remote repository URLs.
git clone <repository-url>       # Copies a remote repository to the local machine.
git fetch origin                 # Downloads remote updates without integrating them into the current branch.
git pull origin main             # Fetches and integrates changes from the remote main branch.
git push origin main             # Pushes local main branch commits to the origin remote.
git merge feature-branch         # Merges the feature branch into the current branch.
git rebase main                  # Replays the current branch's commits on top of main.
git stash                        # Temporarily saves supported uncommitted changes.
git stash list                   # Lists saved stashes.
git stash pop                    # Reapplies the latest stash and removes it if successfully applied.
git stash apply                  # Reapplies the latest stash without removing it.
git cherry-pick <commit-hash>    # Applies a specific commit's changes to the current branch.
git reset --soft HEAD~1          # Moves HEAD back one commit while keeping changes staged.
git reset --mixed HEAD~1         # Moves HEAD back one commit and unstages changes while keeping working files.
git reset --hard HEAD~1          # Resets HEAD, staging, and working files; can discard changes.
git revert <commit-hash>         # Creates a new commit that reverses an earlier commit's changes.
```

### 6. GitHub CLI Commands

```bash
gh auth login                   # Authenticates GitHub CLI with a GitHub account.
gh auth status                  # Displays the current GitHub CLI authentication status.
gh repo list                    # Lists repositories accessible to the authenticated account.
gh repo create                  # Starts repository creation through an interactive prompt.
gh issue list                   # Lists issues in the current repository.
gh issue create                 # Creates an issue through an interactive prompt.
gh pr list                      # Lists pull requests in the current repository.
gh pr create                    # Creates a pull request through an interactive prompt.
gh pr view                      # Displays details of a pull request.
gh repo create <repo-name> --public --source=. --remote=origin --push # Creates a public repository, adds origin, and pushes the current project.
```

### 7. Repository Submission Commands

Run these commands from the repository root, adjusting the file path if necessary.

```bash
git status                                      # Checks which files have changed.
git add 2026/day-28/day-28-notes.md             # Stages the Day 28 Markdown file.
git commit -m "docs: add day 28 revision notes"  # Saves the staged changes as a commit.
git push origin HEAD                            # Pushes the current branch to the origin remote.
```

### 8. Daily Cron Scheduling

```cron
0 3 * * * /bin/bash /path/to/script.sh # Runs the specified Bash script every day at 3:00 AM.
```

The five cron fields represent minute, hour, day of month, month, and day of week.

### Important Safety Notes

- Replace placeholders such as `<PID>`, `<commit-hash>`, `<repository-url>`, and `/path/to/script.sh` with actual values.
- Verify file paths before running `rm` or other destructive commands.
- Use `git reset --hard` carefully because it can discard uncommitted changes.
- Verify the target disk and back up important data before performing LVM operations.
- Use `sudo` only when administrative privileges are required.
- Test commands in a safe environment before using them on production systems.


**Key takeaway:** Git branching helps developers collaborate safely, organize changes, and integrate new features systematically.


