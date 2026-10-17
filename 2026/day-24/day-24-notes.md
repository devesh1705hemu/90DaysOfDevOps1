# Day 24: Advanced Git Concepts

## Task 1: Git Merge

**Goal:** Understand fast-forward merges, merge commits, and merge conflicts.

### 1. Fast-Forward Merge

```bash
git switch main
git switch -c feature-login

echo "Login feature" > login.txt
git add login.txt
git commit -m "Add login feature"

git switch main
git merge feature-login
```

**Use:** Integrates a feature branch into the current branch by moving the branch pointer forward when possible.

### 2. Merge Commit

```bash
git switch main
git switch -c feature-profile

echo "Profile feature" > profile.txt
git add profile.txt
git commit -m "Add profile feature"

git switch main
git merge --no-ff feature-profile
```

**Use:** Creates a merge commit to preserve the feature branch's integration point in Git history.

### 3. Merge Conflict

```bash
git switch main
echo "Hello from main" > message.txt
git add message.txt
git commit -m "Add main message"

git switch -c feature-conflict
echo "Hello from feature" > message.txt
git add message.txt
git commit -m "Update feature message"

git switch main
echo "Hello from updated main" > message.txt
git add message.txt
git commit -m "Update main message"

git merge feature-conflict
```

If Git reports a conflict, open `message.txt`, choose the correct content, and remove the conflict markers.

```bash
git add message.txt
git commit -m "Resolve merge conflict"
git status
```

**Use:** Conflicts occur when Git cannot automatically combine competing changes.

## Task 2: Git Rebase

**Goal:** Replay commits on top of another branch and understand the difference between merge and rebase.

```bash
git switch main
git switch -c feature-rebase

echo "Feature work" > rebase.txt
git add rebase.txt
git commit -m "Add feature work"

git rebase main
```

**Use:** Replays the current branch's commits on top of the latest `main` history.

### Rebase Conflict Commands

```bash
git status
git add <filename>
git rebase --continue
```

| Command | Use |
|---|---|
| `git rebase main` | Replay current branch commits on `main`. |
| `git rebase --continue` | Continue after resolving conflicts. |
| `git rebase --abort` | Cancel the rebase and return to the original state. |
| `git rebase --skip` | Skip the current commit being replayed. |

### Merge vs Rebase

- **Merge:** Preserves branch history and may create a merge commit.
- **Rebase:** Replays commits to create a more linear history.
- **Important:** Avoid rebasing commits that other people already depend on because rebase rewrites commit history.

View the history:

```bash
git log --oneline --graph --all
```

## Task 3: Squash vs Merge Commit

**Goal:** Understand how to combine feature changes and preserve branch history.

### Squash Merge

```bash
git switch main
git merge --squash feature-profile
git commit -m "Add user profile feature"
```

**Use:** Combines a feature branch's changes into one new commit on the current branch without preserving the individual feature commits in the target branch's history.

### Merge Commit

```bash
git merge --no-ff feature-settings
```

**Use:** Creates a merge commit to preserve the feature branch's integration point.

### View Commit History

```bash
git log --oneline
git log --oneline --graph --all
```

**Key difference:** Squash merge creates one commit for the combined changes, while a merge commit records the branch integration and preserves the existing commits.

## Task 4: Git Stash

**Goal:** Temporarily save changes without committing them.

### Save Changes

```bash
git stash
git stash push -m "WIP: dashboard"
git stash push -u -m "WIP: dashboard"
```

**Uses:**
- `git stash` saves tracked-file changes temporarily.
- `git stash push -m "WIP: dashboard"` saves changes with a descriptive message.
- `git stash push -u -m "WIP: dashboard"` also includes untracked files.

### Manage Stashes

```bash
git stash list
git stash show -p
git stash apply
git stash pop
git stash apply 'stash@{1}'
git stash drop 'stash@{1}'
git stash clear
```

| Command | Use |
|---|---|
| `git stash list` | List saved stashes. |
| `git stash show -p` | Display the latest stash's changes. |
| `git stash apply` | Restore the latest stash while keeping it saved. |
| `git stash pop` | Restore the latest stash and remove it if successful. |
| `git stash apply 'stash@{1}'` | Apply a specific stash without removing it. |
| `git stash drop 'stash@{1}'` | Delete a specific stash. |
| `git stash clear` | Delete all stash entries. |

**Real-world example:** Stash unfinished changes before switching branches to work on an urgent bug fix, then restore your changes later.

## Task 5: Git Cherry-Pick

**Goal:** Apply a specific commit from another branch to your current branch.

### Example

```bash
git switch feature-login
git log --oneline
git cherry-pick <commit-hash>
```

Replace `<commit-hash>` with the actual commit ID you want to apply.

### Cherry-Pick Conflict Commands

```bash
git status
git add <filename>
git cherry-pick --continue
```

| Command | Use |
|---|---|
| `git cherry-pick <commit-hash>` | Apply a specific commit to the current branch. |
| `git cherry-pick --continue` | Continue after resolving conflicts. |
| `git cherry-pick --abort` | Cancel the current cherry-pick operation. |
| `git cherry-pick --skip` | Skip the current commit. |
| `git show <commit-hash>` | Inspect a commit and its changes. |
| `git log --oneline -5` | Display the five most recent commits. |

**Real-world example:** Apply a bug fix from a development branch to a release branch without merging all development changes.

## Important Git Commands Reference

| Command | Use |

## Day 24 – Advanced Git Commands

### 1. Git Merge

| Command | Use |
|---|---|
| `git switch main` | Switch to the main branch. |
| `git switch -c feature-login` | Create and switch to a new feature branch. |
| `git merge feature-login` | Merge the feature branch into the current branch. |
| `git merge --no-ff feature-login` | Merge a branch while forcing a merge commit. |
| `git merge --squash feature-profile` | Combine a branch's changes into the staging area without creating a commit. |
| `git merge --abort` | Abort an in-progress merge and restore the pre-merge state when possible. |
| `git status` | Check the working tree and identify merge conflicts. |
| `git log --oneline --graph --all` | Display commit history graphically across all branches. |

### 2. Git Rebase

| Command | Use |
|---|---|
| `git rebase main` | Replay the current branch's commits on top of `main`. |
| `git rebase --continue` | Continue rebasing after resolving conflicts and staging the fixes. |
| `git rebase --abort` | Cancel an in-progress rebase and return to its original state. |
| `git rebase --skip` | Skip the current commit during a rebase. |
| `git log --oneline --graph --all` | Inspect branch history after rebasing. |

### 3. Git Squash and Commit History

| Command | Use |
|---|---|
| `git merge --squash feature-profile` | Prepare all feature branch changes as one combined change on the current branch. |
| `git commit -m "Add user profile feature"` | Create one commit containing the staged squash changes. |
| `git merge --no-ff feature-settings` | Merge a feature branch while retaining a separate merge commit. |
| `git log --oneline` | Display a concise commit history. |
| `git log --oneline --graph --all` | Compare merge and squash history across branches. |

### 4. Git Stash

| Command | Use |
|---|---|
| `git stash` | Temporarily save tracked-file changes and revert them from the working tree. |
| `git stash push -m "WIP: dashboard"` | Save current tracked changes with a descriptive stash message. |
| `git stash push -u -m "WIP: dashboard"` | Stash tracked and untracked files with a descriptive message. |
| `git stash list` | List all saved stashes. |
| `git stash pop` | Restore the latest stash and remove it if application succeeds. |
| `git stash apply` | Restore the latest stash while keeping it in the stash list. |
| `git stash apply 'stash@{1}'` | Apply a specific stash without removing it from the stash list. |
| `git stash drop 'stash@{1}'` | Delete a specific stash entry. |
| `git stash clear` | Delete all stash entries permanently. |
| `git stash show -p` | Display the changes stored in the latest stash. |

### 5. Git Cherry-Pick

| Command | Use |
|---|---|
| `git cherry-pick <commit-hash>` | Apply a specific commit's changes to the current branch as a new commit. |
| `git cherry-pick --continue` | Continue a cherry-pick after resolving conflicts and staging the fixes. |
| `git cherry-pick --abort` | Cancel an in-progress cherry-pick and return to the pre-operation state. |
| `git cherry-pick --skip` | Skip the current commit during a multi-commit cherry-pick operation. |
| `git show <commit-hash>` | Inspect a specific commit's metadata and changes. |
| `git log --oneline -5` | Display the five most recent commits. |

### 6. Conflict Resolution and Inspection

| Command | Use |
|---|---|
| `git status` | Identify conflicted files and the current Git operation's state. |
| `git diff` | Display unstaged changes, including unresolved conflict markers. |
| `git diff --staged` | Review changes staged for the next commit. |
| `git add <filename>` | Stage a file after editing or resolving conflicts. |
| `git commit -m "Resolve merge conflict"` | Commit staged changes after completing a merge conflict resolution. |
| `git switch <branch-name>` | Switch to an existing branch. |
| `git branch` | List local branches and identify the current branch. |

### 7. Essential Advanced Git Reminders

- **Merge:** Integrates branch histories and may create a merge commit when branches diverge.
- **Rebase:** Replays commits on a new base and creates new commit hashes.
- **Squash merge:** Combines feature changes into one new commit on the target branch.
- **Stash:** Temporarily saves unfinished work so you can switch tasks.
- **Cherry-pick:** Applies a selected commit without merging the entire source branch.
- **Conflict resolution:** Edit conflicted files, stage the fixes, and continue or complete the operation.
- **Shared history:** Avoid rebasing commits that others have already pulled unless the rewrite is coordinated.


## Day 24: Key Learnings

- **Merge:** Integrates changes from one branch into another.
- **Rebase:** Replays commits onto a new base to create a linear history.
- **Squash:** Combines multiple changes into one commit.
- **Stash:** Temporarily saves unfinished changes.
- **Cherry-pick:** Applies a selected commit to the current branch.
- **Conflict resolution:** Review conflicting files, edit them, stage the resolved files, and complete the operation.


```
