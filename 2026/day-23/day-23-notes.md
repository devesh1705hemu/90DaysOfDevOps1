# Day 23: Git Branching & Working with GitHub

## Task 1: Understanding Branches

### 1. What is a branch in Git?

A branch in Git is an independent line of development that allows developers to work on new features, bug fixes, and experiments without directly affecting the main codebase.

### 2. Why do we use branches instead of committing everything to `main`?

- **Isolation:** Work on features independently.
- **Safety:** Keep the main branch stable.
- **Collaboration:** Multiple developers can work on different features simultaneously.
- **Code review:** Review and test changes before merging them into `main`.
- **Experimentation:** Try new ideas without affecting the main codebase.

### 3. What is `HEAD` in Git?

`HEAD` is a reference to the currently checked-out commit, usually through the current branch. It tells Git which branch or commit we are currently working from.

### 4. What happens to your files when you switch branches?

Git updates the working directory to match the target branch. Committed changes remain on their original branches. Uncommitted changes may carry over if there are no conflicts, or Git may prevent the switch if it would overwrite those changes.

---

## Task 2: Branching Commands: Hands-On

### 1. List all branches

```bash
git branch
git branch -a
```

**Uses:**
- `git branch`: Lists all local branches.
- `git branch -a`: Lists local and remote-tracking branches.

### 2. Create a new branch

```bash
git branch feature-1
```

**Use:** Creates a new branch named `feature-1` without switching to it.

### 3. Switch to `feature-1`

```bash
git switch feature-1
```

**Use:** Switches the working directory to the existing `feature-1` branch.

### 4. Create and switch to `feature-2`

```bash
git switch -c feature-2
```

**Use:** Creates `feature-2` and switches to it in one command.

### 5. Difference between `git switch` and `git checkout`

```bash
git switch feature-1
git checkout feature-1
git checkout -b feature-3
```

- `git switch feature-1`: Switches to an existing branch.
- `git checkout feature-1`: Switches branches using the older, multifunction command.
- `git checkout -b feature-3`: Creates and switches to a new branch.

`git switch` is designed specifically for branch switching, whereas `git checkout` can also check out files and commits.

### 6. Make a commit on `feature-1`

```bash
git switch feature-1
echo "Feature-1 test" > feature-1.txt
git add feature-1.txt
git commit -m "Add feature-1 test file"
```

**Uses:**
- `git switch feature-1`: Switches to the feature branch.
- `echo "Feature-1 test" > feature-1.txt`: Creates a file containing test text.
- `git add feature-1.txt`: Stages the file.
- `git commit -m "Add feature-1 test file"`: Commits the staged changes.

### 7. Verify that the commit does not exist on `main`

```bash
git switch main
git log --oneline main
git log --oneline feature-1
git diff main..feature-1
```

**Uses:**
- `git switch main`: Switches back to `main`.
- `git log --oneline main`: Displays the commit history of `main`.
- `git log --oneline feature-1`: Displays the commit history of `feature-1`.
- `git diff main..feature-1`: Shows the differences between the branch tips.

The commit created on `feature-1` should not appear in `main` until the feature branch is merged.

### 8. Compare files between branches

```bash
git ls-tree -r --name-only main
git ls-tree -r --name-only feature-1
```

**Use:** Lists tracked files in each branch so you can compare their contents.

### 9. Delete a branch

```bash
git branch -d feature-2
```

**Use:** Deletes a branch that has been safely merged.

**Note:** Git normally refuses to delete an unmerged branch with `-d`. Do not delete `feature-1` while you still need its unmerged commit.

### 10. Check the current branch and status

```bash
git branch
git status
```

**Uses:**
- `git branch`: Shows local branches and marks the current one.
- `git status`: Displays the current branch, staged changes, and working-tree status.

---

## Task 3: Push to GitHub

### 1. Connect the local repository to GitHub

```bash
git remote -v
git remote add origin https://github.com/YOUR-USERNAME/devops-git-practice.git
```

**Uses:**
- `git remote -v`: Displays configured remote repositories.
- `git remote add origin URL`: Adds the GitHub repository as the `origin` remote.

Replace `YOUR-USERNAME` with your GitHub username. If `origin` already exists, use `git remote set-url origin URL` to update it.

### 2. Push `main` to GitHub

```bash
git switch main
git push -u origin main
```

**Uses:**
- `git switch main`: Switches to the main branch.
- `git push -u origin main`: Pushes `main` to GitHub and sets its upstream tracking relationship.

### 3. Push `feature-1` to GitHub

```bash
git switch feature-1
git push -u origin feature-1
```

**Use:** Publishes the feature branch to GitHub and configures tracking.

### 4. Verify remote branches

```bash
git branch -a
git branch -r
git ls-remote --heads origin
```

**Uses:**
- `git branch -a`: Lists local and remote-tracking branches.
- `git branch -r`: Lists remote-tracking branches.
- `git ls-remote --heads origin`: Lists branch references available on the remote repository.

### 5. Difference between `origin` and `upstream`

- **Origin:** The conventional name for your primary remote repository, often your own GitHub repository or fork.
- **Upstream:** Commonly the name given to the original repository when you fork someone else's project.

These are conventional remote names, not special Git keywords.

---

## Task 4: Pull from GitHub

### 1. Make a change directly on GitHub

Open `git-commands.md` on GitHub, click the edit icon, add a line, and commit the change to `main`.

### 2. Fetch changes from GitHub

```bash
git fetch origin
```

**Use:** Downloads remote updates without integrating them into the current branch.

### 3. Check commits available remotely

```bash
git log --oneline main..origin/main
```

**Use:** Displays commits on remote-tracking `origin/main` that are not yet in local `main`.

### 4. Pull changes into the local repository

```bash
git switch main
git pull origin main
```

**Uses:**
- `git switch main`: Switches to the local main branch.
- `git pull origin main`: Fetches and integrates changes from the remote main branch.

### 5. Verify the updated file

```bash
cat git-commands.md
tail -n 20 git-commands.md
git status
```

**Uses:**
- `cat git-commands.md`: Displays the complete file.
- `tail -n 20 git-commands.md`: Displays the last 20 lines.
- `git status`: Checks the working-tree state and synchronization status.

### 6. Difference between `git fetch` and `git pull`

**Git fetch:** Downloads remote changes and updates remote-tracking references without integrating those changes into the current branch.

**Git pull:** Fetches remote changes and integrates them into the current branch, typically through a fast-forward or merge, or a rebase if configured.

Use `git fetch` when you want to inspect remote changes before integrating them. Use `git pull` when you are ready to integrate them.

---

## Task 5: Clone vs Fork

### 1. Clone a public repository

```bash
git clone https://github.com/progit/progit2.git
```

**Use:** Downloads the public Pro Git repository to your local machine.

### 2. Fork the same repository

Open https://github.com/progit/progit2 and click **Fork** to create a copy under your GitHub account.

### 3. Clone your fork

```bash
git clone https://github.com/YOUR-USERNAME/progit2.git progit2-fork
cd progit2-fork
```

**Uses:**
- `git clone URL progit2-fork`: Clones your fork into a directory named `progit2-fork`.
- `cd progit2-fork`: Enters the cloned repository.

### 4. Difference between clone and fork

**Clone:** Creates a local copy of a repository on your computer.

**Fork:** Creates a separate GitHub-hosted copy of another user's repository under your account.

A clone is useful for local development, testing, and exploring code. A fork is useful when you want to contribute to someone else's project without directly modifying their repository.

### 5. Keep your fork in sync with the original repository

Add the original repository as `upstream`:

```bash
git remote add upstream https://github.com/progit/progit2.git
```

**Use:** Adds the original project as another remote.

Fetch the original repository's latest updates:

```bash
git fetch upstream
```

**Use:** Downloads changes from the original repository without integrating them.

Check the original repository's default branch:

```bash
git remote show upstream
git branch -a
```

**Uses:**
- `git remote show upstream`: Displays information about the upstream remote and its branches.
- `git branch -a`: Lists local and remote-tracking branches.

Switch to the branch you want to update and merge the original branch. For example, if the original branch is `master`:

```bash
git switch master
git merge upstream/master
```

If the original branch is `main`, use:

```bash
git switch main
git merge upstream/main
```

**Use:** Integrates changes from the original repository into your local branch.

Push the updated branch to your fork:

```bash
git push origin master
```

Use `git push origin main` instead if you updated `main`.

**Use:** Publishes your updated branch to your fork on GitHub.

---

## Key Learnings

1. Branches allow isolated development without immediately affecting `main`.
2. `HEAD` identifies the currently checked-out commit or branch.
3. `git switch` is a clear way to change branches.
4. `git push` publishes local commits to a remote repository.
5. `origin` commonly refers to your primary remote, while `upstream` commonly refers to the original project.
6. `git fetch` downloads updates, while `git pull` downloads and integrates them.
7. Cloning creates a local copy; forking creates a separate GitHub-hosted copy.
8. You can keep a fork synchronized by fetching from `upstream`, merging the original branch, and pushing to `origin`.

