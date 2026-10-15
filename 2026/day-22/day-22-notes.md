# Day 22: Introduction to Git
## Your First Repository

**Challenge:** #90DaysOfDevOps  
**Topic:** Git and Version Control  


---

## 1. Overview

Git is a distributed version control system that tracks changes in files and helps developers collaborate on projects.

Git is an essential DevOps skill because it supports source code management, collaboration, automation, and CI/CD workflows.

### Learning Objectives

- Understand Git and why version control matters.
- Install and configure Git.
- Create a repository from scratch.
- Understand the working directory, staging area, and repository.
- Create and maintain a Git commands reference.
- Stage, commit, and inspect changes.
- Build a meaningful commit history.
- Learn to use Git documentation and help commands.

---

## 2. Task 1: Install and Configure Git

### Objective
Verify Git installation and configure the author identity.

### Commands

```bash
git --version
git config --global user.name "Devesh Arya"
git config --global user.email "your-email@example.com"
git config --list
```

### Expected Outcome

- Git is installed and its version is displayed.
- Git username and email are configured.
- Configuration settings are verified.

**Note:** Replace the example email with your actual email address.

---

## 3. Task 2: Create Your Git Project

### Objective
Create and initialize a local Git repository.

### Commands

```bash
mkdir devops-git-practice
cd devops-git-practice
git init
git status
pwd
ls -la
ls -la .git
cat .git/HEAD
```

### What I Learned

- `mkdir` creates a directory.
- `cd` changes the current directory.
- `git init` initializes a Git repository.
- `git status` reports the state of the repository.
- `pwd` displays the current directory path.
- `ls -la` lists files, including hidden files.
- `.git/` contains Git's internal repository data.
- `.git/HEAD` identifies the current branch or commit reference.

---

## 4. Task 3: Create a Git Commands Reference

### Objective
Maintain a reusable reference of Git commands in `git-commands.md`.

### Required Categories

1. Setup & Config
2. Basic Workflow
3. Viewing Changes

### Commands to Document

| Command | Purpose |
|---|---|
| `git --version` | Displays the installed Git version. |
| `git config --list` | Displays Git configuration settings. |
| `git init` | Initializes a Git repository. |
| `git status` | Shows repository and file status. |
| `git add` | Stages changes for a commit. |
| `git diff` | Shows unstaged changes to tracked files. |
| `git diff --cached` | Shows staged changes. |
| `git commit` | Records staged changes in repository history. |
| `git log` | Displays commit history. |
| `git log --oneline` | Displays compact commit history. |

For every command, document its purpose and provide an example.

---

## 5. Task 4: Stage and Commit

### Objective
Save the Git commands reference in repository history.

### Commands

```bash
git add git-commands.md
git status
git diff --cached
git commit -m "Add Git commands reference"
git log
```

### What I Learned

- Staging selects changes for the next commit.
- `git diff --cached` lets me review staged changes.
- A commit records a snapshot in local repository history.
- `git log` displays saved commits.

---

## 6. Task 5: Make More Changes and Build History

### Objective
Update `git-commands.md` and create at least three additional commits.

### Workflow

```bash
git diff
git add git-commands.md
git diff --cached
git commit -m "Document Git staging and diff commands"

git diff
git add git-commands.md
git diff --cached
git commit -m "Document Git commits and history commands"

git diff
git add git-commands.md
git diff --cached
git commit -m "Document Git branches and repository commands"
```

### View Compact History

```bash
git log --oneline
git log --oneline --graph --decorate --all
```

### What I Learned

- Small, focused commits make history easier to understand.
- Commit messages should explain the changes.
- Git history helps track how a project evolves.
- Reviewing changes before staging and committing reduces mistakes.

**Expected Outcome:** At least three additional commits beyond the original reference-file commit.

---

## 7. Task 6: Understand the Git Workflow

### Question 1: What is the difference between `git add` and `git commit`?

`git add` stages the changes I want to include in a commit. `git commit` saves those staged changes as a snapshot in the repository history.

### Question 2: What does the staging area do?

The staging area lets me select exactly which changes should be included in the next commit. It gives me a chance to review changes before saving them.

### Question 3: What information does `git log` show?

`git log` displays the commit history, including commit IDs, authors, dates, and commit messages.

### Question 4: What is the `.git/` folder?

The `.git/` directory stores Git metadata, configuration, objects, references, and repository history. Deleting it generally removes the repository's local version-control metadata, although project files may remain.

### Question 5: What is the difference between the working directory, staging area, and repository?

- **Working directory:** Where I create and edit project files.
- **Staging area:** Where I select changes for the next commit.
- **Repository:** Where Git stores committed snapshots and history.

The workflow is:

Working Directory -> Staging Area -> Repository

### Question 6: Why is `git status` important?

`git status` shows the current branch and whether files are untracked, modified, staged, or ready to commit. It helps me understand what Git will include in my next commit.

### Question 7: How can I explore Git help?

I can use Git's built-in documentation to learn command options and usage.

Examples:

```bash
git status --help
git add --help
git commit --help
git log --help
```

On Linux, these commands may open a manual viewer. Press `q` to exit. If a manual viewer is unavailable, try:

```bash
git status -h
git add -h
```

---

## 8. Understanding the Git Workflow

```text
Working Directory
       |
       | git add
       v
Staging Area
       |
       | git commit
       v
Local Repository
       |
       | git log
       v
Commit History
```

---

## 9. Final Verification

Run the following commands:

```bash
git status
git log --oneline
git diff HEAD~3 HEAD --stat
```

## 10. Git and Linux Commands: One-Line Uses

| Command | One-line use | Example |
|---|---|---|
| `git --version` | Checks the installed Git version. | `git --version` |
| `git config --global user.name` | Sets your name for Git commits. | `git config --global user.name "Devesh Arya"` |
| `git config --global user.email` | Sets your email for Git commits. | `git config --global user.email "your-email@example.com"` |
| `git config --list` | Displays configured Git settings. | `git config --list` |
| `mkdir` | Creates a new directory. | `mkdir devops-git-practice` |
| `cd` | Changes the current working directory. | `cd devops-git-practice` |
| `pwd` | Prints the current directory path. | `pwd` |
| `ls -la` | Lists all files, including hidden files. | `ls -la` |
| `touch` | Creates an empty file if it does not exist. | `touch git-commands.md` |
| `git init` | Initializes a Git repository in the current directory. | `git init` |
| `git status` | Shows the current branch and file changes. | `git status` |
| `cat` | Displays the contents of a file. | `cat git-commands.md` |
| `cat > file.md` | Writes content to a file, replacing existing content. | `cat > notes.md` |
| `cat >> file.md` | Appends content to the end of a file. | `cat >> notes.md` |
| `ls -la .git` | Lists the contents of Git's internal directory. | `ls -la .git` |
| `cat .git/HEAD` | Displays the current branch or commit reference. | `cat .git/HEAD` |
| `git add` | Stages selected file changes for the next commit. | `git add git-commands.md` |
| `git diff` | Shows unstaged changes in tracked files. | `git diff` |
| `git diff --cached` | Shows changes currently staged for committing. | `git diff --cached` |
| `git commit -m` | Saves staged changes as a commit with a message. | `git commit -m "Update Git notes"` |
| `git log` | Displays detailed commit history. | `git log` |
| `git log --oneline` | Displays each commit on a single line. | `git log --oneline` |
| `git log --oneline --graph --decorate --all` | Displays commit history with a graph and branch labels. | `git log --oneline --graph --decorate --all` |
| `git show HEAD` | Shows details and changes from the latest commit. | `git show HEAD` |
| `git branch` | Lists local branches. | `git branch` |
| `git branch <branch-name>` | Creates a new branch without switching to it. | `git branch feature/docs` |
| `git branch --show-current` | Prints the current branch name. | `git branch --show-current` |
| `git rev-parse --show-toplevel` | Displays the root directory of the current repository. | `git rev-parse --show-toplevel` |
| `git remote -v` | Lists configured remote repositories and their URLs. | `git remote -v` |
| `git status --help` | Opens help documentation for the status command. | `git status --help` |
| `git add -h` | Displays brief help for the add command. | `git add -h` |

### Important Notes

- `git add` stages changes; it does not create a commit.
- `git commit` records staged changes in local repository history.
- `git diff` normally shows unstaged changes to tracked files, not every untracked file.
- `git status` is useful for checking what is staged, modified, or untracked.
- `.git/` stores repository metadata; deleting it can remove local Git history and configuration.
- `git remote -v` only lists remotes; it does not upload code.
- Replace example email addresses with your actual Git email where appropriate.

