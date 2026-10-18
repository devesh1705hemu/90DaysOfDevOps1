# Day 25 – Git Reset vs Revert & Branching Strategies

## Objective

Today, I learned how to undo mistakes safely using Git Reset and Git Revert. I also explored GitFlow, GitHub Flow, and Trunk-Based Development, which are used by engineering teams to manage code and collaborate effectively.

## Task 1: Git Reset – Hands-On

### 1. Create Three Commits

Create a practice file and make three commits:

```bash
echo "Commit A - Initial setup" > reset-practice.txt
git add reset-practice.txt
git commit -m "Commit A: Initial setup"

echo "Commit B - Add configuration" >> reset-practice.txt
git add reset-practice.txt
git commit -m "Commit B: Add configuration"

echo "Commit C - Add documentation" >> reset-practice.txt
git add reset-practice.txt
git commit -m "Commit C: Add documentation"

git log --oneline -3
```

### 2. Git Reset Soft

```bash
git reset --soft HEAD~1
git status
git diff --cached
```

**Observation:** `git reset --soft HEAD~1` moves the current branch pointer back by one commit while preserving changes in the staging area. The changes from the removed commit remain staged and can be committed again.

### 3. Git Reset Mixed

After re-committing the staged changes, run:

```bash
git commit -m "Recommit after soft reset"
git reset --mixed HEAD~1
git status
git diff
```

**Observation:** `git reset --mixed HEAD~1` moves the branch pointer back by one commit and resets the staging area. The affected changes remain in the working directory as unstaged changes.

### 4. Git Reset Hard

After staging and committing the changes again, run:

```bash
git add reset-practice.txt
git commit -m "Recommit after mixed reset"
git reset --hard HEAD~1
git status
```

**Observation:** `git reset --hard HEAD~1` moves the branch pointer back, resets the staging area, and resets tracked working files to the target commit. Changes discarded by this operation may be difficult or impossible to recover.

**Important:** Run these experiments only in a practice repository. Check `git status` before using `--hard`.

### 5. Answers

**Q1. What is the difference between `--soft`, `--mixed`, and `--hard`?**

- `--soft`: Moves HEAD and preserves staged and working-directory changes.
- `--mixed`: Moves HEAD, resets the staging area, and preserves working-directory changes. This is the default reset mode.
- `--hard`: Moves HEAD and resets both the staging area and tracked working files to the target commit.

**Q2. Which one is destructive and why?**

`git reset --hard` is destructive because it discards affected tracked working-tree changes and can remove uncommitted work. Recovery is not guaranteed.

**Q3. When would you use each one?**

- `--soft`: When I want to undo a commit but keep its changes staged for another commit.
- `--mixed`: When I want to undo a commit and review or edit its changes before staging again.
- `--hard`: When I intentionally want to discard changes and return tracked files to a particular commit.

**Q4. Should I use `git reset` on commits that are already pushed?**

Generally, I should avoid resetting commits that have already been shared with others because it can rewrite branch history and require a force push. For shared branches, `git revert` is usually safer.

---

## Task 2: Git Revert – Hands-On

### 1. Create Three Commits

```bash
echo "Commit X - Initial setup" > revert-practice.txt
git add revert-practice.txt
git commit -m "Commit X: Initial setup"

echo "Commit Y - Add configuration" >> revert-practice.txt
git add revert-practice.txt
git commit -m "Commit Y: Add configuration"

echo "Commit Z - Add documentation" >> revert-practice.txt
git add revert-practice.txt
git commit -m "Commit Z: Add documentation"

git log --oneline -3
```

### 2. Revert the Middle Commit

First, identify the hash of Commit Y:

```bash
git log --oneline
```

Copy the hash belonging to `Commit Y: Add configuration`, then run:

```bash
git revert <commit-Y-hash>
```

If Git reports a conflict:

```bash
git status
cat revert-practice.txt
```

Resolve the conflict according to the intended final contents. If the goal is to remove Y's configuration line while keeping X and Z, the file should contain:

```text
Commit X - Initial setup
Commit Z - Add documentation
```

Then complete the revert:

```bash
git add revert-practice.txt
git revert --continue
```

If I want to cancel the operation instead, I can use:

```bash
git revert --abort
```

### 3. Check the History

```bash
git log --oneline
git show <revert-commit-hash>
cat revert-practice.txt
```

**Observation:** Git Revert creates a new commit that reverses the selected commit's changes. The original Commit Y remains in the history, and the new revert commit records the reversal.

The exact final file contents depend on the changes in each commit and whether conflicts need manual resolution.

### 4. Answers

**Q1. How is `git revert` different from `git reset`?**

`git reset` moves the current branch pointer and may change the staging area or working directory. `git revert` creates a new commit that reverses changes introduced by an earlier commit.

**Q2. Why is revert safer for shared branches?**

It preserves the existing commit history and records the undo operation as a new commit. Other collaborators do not need to reconcile a rewritten branch history.

**Q3. When should I use revert vs reset?**

- Use `git revert` to undo changes already pushed or shared with a team.
- Use `git reset` to reorganize or undo local, unpublished commits when appropriate.

---

## Task 3: Reset vs Revert – Summary

| Comparison | `git reset` | `git revert` |
|---|---|---|
| What it does | Moves the current branch pointer and may reset the index or working tree | Creates a new commit that reverses another commit's changes |
| Removes commit from history? | The commit may no longer be reachable from the current branch | The original commit remains in history |
| Safe for shared/pushed branches? | Usually avoid rewriting shared history | Generally safer for shared branches |
| Creates a new commit? | No | Yes |
| When to use | Local, unpublished history changes | Undoing changes in shared or pushed history |

### Key Learning

Git Reset changes where the branch points. Git Revert preserves the original history and adds a new commit to reverse changes.

---

## Task 4: Branching Strategies

### 1. GitFlow

**How it works:** GitFlow uses dedicated branches for development, feature work, release preparation, and production hotfixes.

**Branches:**

- `main`: Holds production-ready releases.
- `develop`: Integrates upcoming development changes.
- `feature/*`: Develops new features.
- `release/*`: Stabilizes a planned release.
- `hotfix/*`: Fixes urgent production issues.

**Text-based flow:**

```text
feature/* -> develop -> release/* -> main
hotfix/* -----------------------> main
hotfix/* -----------------------> develop
```

**When it is used:** Products with scheduled releases, formal testing, and version maintenance.

**Pros:**
- Clear release management.
- Separates development and production workflows.
- Provides a dedicated hotfix process.

**Cons:**
- More branches to maintain.
- Additional merging and synchronization.
- Can slow down frequent deployment.

### 2. GitHub Flow

**How it works:** Developers create short-lived feature branches from `main`, make changes, open pull requests, complete code review and automated tests, and merge approved changes.

**Text-based flow:**

```text
main
 |
 v
feature branch
 |
 v
pull request
 |
 v
code review + automated tests
 |
 v
main
 |
 v
deploy
```

**When it is used:** Startups, web applications, SaaS products, and teams shipping frequently.

**Pros:**
- Simple and easy to understand.
- Encourages code review.
- Works well with CI/CD.

**Cons:**
- Requires reliable testing.
- Incomplete features may need feature flags.
- Complex release processes may need additional controls.

### 3. Trunk-Based Development

**How it works:** Developers integrate small changes into a shared `main` or trunk branch frequently. Teams may use short-lived branches, but they merge them quickly.

**Text-based flow:**

```text
Small developer changes
          |
          v
Short-lived branches or direct commits
          |
          v
        main
          |
          v
Automated CI and tests
          |
          v
       Deploy
```

**When it is used:** Teams practicing continuous integration, frequent delivery, and automated testing.

**Pros:**
- Reduces long-lived branch conflicts.
- Gives developers fast feedback.
- Supports frequent integration and delivery.

**Cons:**
- Requires reliable automated tests.
- Requires small, incremental changes.
- Feature flags and backward-compatible changes may be necessary.

### 4. Which Strategy Should I Choose?

**Q1. Which strategy would I use for a startup shipping fast?**

I would choose GitHub Flow because it is simple, supports pull requests, and works well with frequent releases. Trunk-Based Development is also a strong option when automated testing and CI/CD are mature.

**Q2. Which strategy would I use for a large team with scheduled releases?**

I would consider GitFlow when the project requires dedicated release stabilization, version maintenance, and formal release management. The final choice would depend on the team's deployment and testing practices.

**Q3. Which strategy does an open-source project use?**

Example: Kubernetes.

- Repository: https://github.com/kubernetes/kubernetes
- Contributor workflow: https://www.kubernetes.dev/docs/guide/github-workflow/

The Kubernetes contribution workflow uses working branches, pull requests, and code review. This resembles a pull-request-based feature-branch workflow. It should not automatically be treated as proof that the entire project follows pure GitHub Flow.

---

## Task 5: Git Commands Reference Update

The `git-commands.md` file should contain the important commands covered during Days 22–25. The following tables list each command with its use in one line.

### 1. Setup and Configuration

| Command | Use |
|---|---|
| `git --version` | Check the installed Git version. |
| `git config --global user.name "Your Name"` | Set the global Git username. |
| `git config --global user.email "you@example.com"` | Set the global Git email. |
| `git config --list` | Display Git configuration settings. |
| `git config --global init.defaultBranch main` | Set the default initial branch to `main`. |
| `git init` | Initialize a new Git repository. |
| `git clone <repo-url>` | Copy a remote repository to the local machine. |
| `git help <command>` | Display documentation for a Git command. |

### 2. Basic Workflow

| Command | Use |
|---|---|
| `git status` | Display working-tree and staging-area status. |
| `git status -sb` | Display concise status and branch information. |
| `git add <file>` | Stage a specific file. |
| `git add .` | Stage changes under the current directory. |
| `git add -p` | Interactively stage selected changes. |
| `git commit -m "message"` | Save staged changes in a new commit. |
| `git commit --amend` | Modify the latest commit. |
| `git log` | Display detailed commit history. |
| `git log --oneline` | Display compact commit history. |
| `git log --oneline --graph --all` | Visualize reachable commit history and branches. |
| `git show <commit>` | Display details and changes for a commit. |
| `git diff` | Show unstaged changes. |
| `git diff --cached` | Show staged changes. |
| `git diff HEAD` | Compare working-tree changes against `HEAD`. |
| `git restore <file>` | Discard unstaged changes in a tracked file. |
| `git restore --staged <file>` | Unstage a file while preserving its changes. |
| `git rm <file>` | Remove a tracked file and stage its deletion. |
| `git mv <old> <new>` | Move or rename a tracked file. |

### 3. Branching

| Command | Use |
|---|---|
| `git branch` | List local branches. |
| `git branch -a` | List local and remote-tracking branches. |
| `git branch <name>` | Create a branch. |
| `git branch -d <name>` | Delete a branch that has been merged. |
| `git branch -D <name>` | Force-delete a local branch. |
| `git branch -m <new-name>` | Rename the current branch. |
| `git switch <branch>` | Switch to an existing branch. |
| `git switch -c <name>` | Create and switch to a new branch. |
| `git checkout <branch>` | Switch to an existing branch using the older command. |
| `git checkout -b <name>` | Create and switch to a new branch using `checkout`. |
| `git merge <branch>` | Merge the specified branch into the current branch. |

### 4. Remote Repositories and GitHub

| Command or Action | Use |
|---|---|
| `git remote -v` | Display configured remote repository URLs. |
| `git remote add origin <url>` | Add a remote named `origin`. |
| `git remote set-url origin <url>` | Change the URL of the `origin` remote. |
| `git remote remove <name>` | Remove a configured remote. |
| `git clone <url>` | Download a repository and its history. |
| `git fetch origin` | Download remote updates without integrating them. |
| `git pull` | Fetch and integrate remote changes into the current branch. |
| `git pull --rebase` | Fetch remote changes and rebase local commits onto them. |
| `git push` | Push commits to the configured upstream branch. |
| `git push origin <branch>` | Push a specific branch to the remote. |
| `git push -u origin <branch>` | Push a branch and set its upstream. |
| `git push --delete origin <branch>` | Delete a branch from the remote repository. |
| `git push --force-with-lease` | Force-push while checking for unexpected remote updates. |
| Fork a repository on GitHub | Create a copy of a repository under your GitHub account. |
| Open a pull request on GitHub | Request review and merging of your changes. |

### 5. Merging and Rebasing

| Command | Use |
|---|---|
| `git merge <branch>` | Merge another branch into the current branch. |
| `git merge --no-ff <branch>` | Create a merge commit even when fast-forwarding is possible. |
| `git merge --abort` | Cancel an in-progress merge. |
| `git rebase main` | Replay the current branch's commits on top of `main`. |
| `git rebase -i HEAD~3` | Interactively edit the last three commits. |
| `git rebase --continue` | Continue a rebase after resolving conflicts. |
| `git rebase --abort` | Cancel an in-progress rebase. |
| `git rebase --skip` | Skip the commit currently being replayed. |
| `git status` | Identify merge or rebase conflicts. |

### 6. Stash

| Command | Use |
|---|---|
| `git stash` | Temporarily save tracked working-tree and staged changes. |
| `git stash push -m "message"` | Save changes with a descriptive stash message. |
| `git stash -u` | Stash changes including untracked files. |
| `git stash list` | List saved stashes. |
| `git stash show -p` | Display the latest stash's patch. |
| `git stash apply` | Apply the latest stash while keeping it saved. |
| `git stash apply stash@{0}` | Apply a specific stash without deleting it. |
| `git stash pop` | Apply the latest stash and remove it if successful. |
| `git stash drop stash@{0}` | Delete a specific stash entry. |
| `git stash clear` | Remove all stash entries. |

### 7. Cherry-Pick

| Command | Use |
|---|---|
| `git cherry-pick <commit>` | Apply a specific commit to the current branch. |
| `git cherry-pick <commit1> <commit2>` | Apply multiple selected commits. |
| `git cherry-pick --continue` | Continue after resolving a cherry-pick conflict. |
| `git cherry-pick --abort` | Cancel an in-progress cherry-pick. |
| `git cherry-pick --skip` | Skip the current cherry-picked commit. |

### 8. Reset

| Command | Use |
|---|---|
| `git reset --soft HEAD~1` | Undo the latest commit while keeping its changes staged. |
| `git reset --mixed HEAD~1` | Undo the latest commit while keeping its changes unstaged. |
| `git reset --hard HEAD~1` | Move `HEAD` back and reset the index and tracked working files. |
| `git reset HEAD <file>` | Unstage a file while preserving working-tree changes. |
| `git reflog` | Inspect recent local reference movements. |

### 9. Revert

| Command | Use |
|---|---|
| `git revert <commit>` | Create a new commit that reverses an existing commit. |
| `git revert HEAD` | Revert the latest commit. |
| `git revert HEAD~1` | Revert the commit one parent behind `HEAD`. |
| `git revert --continue` | Complete a revert after resolving conflicts. |
| `git revert --abort` | Cancel an in-progress revert. |
| `git revert --skip` | Skip the current revert operation. |

### 10. Recovery and Inspection

| Command | Use |
|---|---|
| `git reflog --all` | Inspect recent local reference movements across references. |
| `git branch -vv` | Display branches and upstream tracking status. |
| `git diff main...feature` | Compare changes on `feature` since the branches diverged from `main`. |
| `git log --all --decorate --oneline --graph` | Visualize reachable commits and branches. |
| `git fsck --lost-found` | Find dangling or unreachable Git objects for recovery investigation. |

### Append the Reference to `git-commands.md`

Use `>>` to append content without overwriting the existing file. Use `>` only when intentionally replacing the file.

```bash
cat << 'EOF' >> git-commands.md

## Day 25: Reset, Revert and Branching Strategies

- `git reset --soft HEAD~1` - Undo the latest commit and keep changes staged.
- `git reset --mixed HEAD~1` - Undo the latest commit and keep changes unstaged.
- `git reset --hard HEAD~1` - Reset HEAD, staging area, and tracked working files to the previous commit.
- `git revert <commit>` - Create a new commit that reverses an existing commit.
- `git reflog` - Inspect recent local reference movements.
- `git branch` - List local branches.
- `git switch -c <name>` - Create and switch to a new branch.
- `git merge <branch>` - Merge another branch into the current branch.
- `git rebase main` - Replay the current branch's commits on top of main.
- `git stash` - Temporarily save tracked changes.
- `git cherry-pick <commit>` - Apply a selected commit to the current branch.

### Branching Strategies

- GitFlow - Uses main, develop, feature, release, and hotfix branches for structured release management.
- GitHub Flow - Uses short-lived feature branches, pull requests, review, and merging into main.
- Trunk-Based Development - Integrates small changes into main frequently using short-lived branches or direct commits.

EOF
```

**Note:** If you already appended this section to `git-commands.md`, do not run the append command again, or you will create duplicate content.

---

## Final Learnings

1. I learned how to undo local commits using soft, mixed, and hard reset.
2. I learned how to reverse existing commits safely using Git Revert.
3. I understood why reverting is generally safer than rewriting shared history.
4. I explored GitFlow, GitHub Flow, and Trunk-Based Development.
5. I updated my Git commands reference with one-line explanations.
6. I strengthened my understanding of Git workflows used in collaborative software development and DevOps.

## Useful Commands to Verify My Work

```bash
git status
git log --oneline --graph --all
git diff -- day-25-notes.md
git diff -- git-commands.md
```

**Day 25 takeaway:** Knowing how to undo mistakes, resolve conflicts, and select an appropriate branching strategy is essential for safe collaboration and reliable CI/CD workflows.
