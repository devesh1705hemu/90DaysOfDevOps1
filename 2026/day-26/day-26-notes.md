# Day 26: GitHub CLI, Manage GitHub from Your Terminal

## Objective

Learn to manage GitHub repositories, issues, pull requests, workflows, and releases from the terminal using GitHub CLI (`gh`). This helps DevOps engineers automate development workflows and reduce dependence on the browser.

## Environment

- **Machine:** AWS EC2
- **Operating system:** Ubuntu Linux
- **GitHub username:** `yourusername`
- **Git operations protocol:** HTTPS
- **Default branch:** `master`
- **CLI tool:** GitHub CLI (`gh`)

---

## Task 1: Install and Authenticate

### Installation

```bash
sudo apt update
sudo apt install -y gh
gh --version
```

### Authentication

```bash
gh auth login
```

Selected options:

- Account: GitHub.com
- Git protocol: HTTPS
- Authenticate Git with GitHub credentials: Yes
- Authentication method: Login with a web browser

Because EC2 is a headless server, I opened the GitHub device authentication URL on my local computer and completed authorization there.

### Verification

```bash
gh auth status
gh api user --jq '.login'
```

### Observation

GitHub CLI was authenticated successfully with my GitHub account. The `gh auth status` command displays the active account, authentication method details, and token scopes.

### Authentication methods supported by GitHub CLI

- Browser-based authentication using a web or device-code flow.
- Personal Access Token (PAT) authentication using `gh auth login --with-token`.
- Environment-variable authentication using `GH_TOKEN` or `GITHUB_TOKEN`, commonly used in scripts and CI/CD pipelines.

---

## Task 2: Working with Repositories

### 1. Create a public repository with a README

```bash
gh repo create gh-cli-practice --public --add-readme --clone
```

This creates a public repository with a README and clones it to the local machine.

### 2. Clone a repository using GitHub CLI

```bash
gh repo clone cli/cli
```

This clones the official GitHub CLI repository without using `git clone` directly.

### 3. View repository details

```bash
gh repo view
gh repo view devesh1705hemu/velloe-finora
```

### 4. List repositories

```bash
gh repo list devesh1705hemu --limit 1000
```

### 5. Open a repository in a browser

```bash
gh repo view --web
```

On a headless EC2 instance, the graphical browser may not be available. To retrieve the URL instead:

```bash
gh repo view --json url --jq '.url'
```

I can copy the printed URL and open it in Chrome on my local computer.

### 6. Delete the test repository

```bash
gh repo delete devesh1705hemu/gh-cli-practice
```

The command requires confirmation. Repository deletion also requires the appropriate account permissions and token authorization.

My initial deletion attempt failed because the token lacked the `delete_repo` scope. The CLI identified the missing scope.

To request the scope:

```bash
gh auth refresh -h github.com -s delete_repo
```

Then retry the deletion only after confirming the exact repository name.

### Observation

GitHub CLI supports repository creation, cloning, inspection, listing, and deletion directly from the terminal. Repository permissions and authentication scopes are separate requirements.

---

## Task 3: Issues

### 1. Create an issue

First, check available labels:

```bash
gh label list
```

If the `enhancement` label does not exist, create it:

```bash
gh label create enhancement \
  --description "Improvements and optimizations" \
  --color "A2EEEF"
```

Create an issue:

```bash
gh issue create \
  --title "Improve Docker image size" \
  --body "Review the Dockerfile and identify opportunities to reduce image size using multi-stage builds and optimized base images." \
  --label "enhancement"
```

### 2. List open issues

```bash
gh issue list --state open
```

### 3. View an issue

Replace `ISSUE_NUMBER` with the actual issue number:

```bash
gh issue view ISSUE_NUMBER
```

### 4. Close an issue

```bash
gh issue close ISSUE_NUMBER \
  --comment "Completed the review and closing this issue."
```

### 5. How can `gh issue` be used in automation?

GitHub CLI can automate issue creation when CI/CD pipelines fail, create maintenance tickets, list unresolved bugs, and add comments to issues. Bash scripts can combine `gh issue` with test results or monitoring alerts to create actionable work items automatically.

### Observation

GitHub issues help track bugs, enhancements, and maintenance work. Using `gh issue` makes it possible to manage this work programmatically.

---

## Task 4: Pull Requests

### 1. Create a branch

```bash
git switch -c docs/add-devops-notes
```

### 2. Make a change

```bash
cat > devops-notes.md <<'EOF'
# DevOps Practice

## GitHub CLI

- Create and manage repositories using gh.
- Track issues and pull requests from the terminal.
- Use pull requests to review changes before merging.
EOF
```

### 3. Commit and push

```bash
git add devops-notes.md
git commit -m "docs: add DevOps practice notes"
git push -u origin docs/add-devops-notes
```

### 4. Create a pull request

My repository uses `master` as the base branch.

```bash
gh pr create \
  --base master \
  --head docs/add-devops-notes \
  --title "Add DevOps practice notes" \
  --body "Adds documentation covering GitHub CLI repository, issue, and pull request workflows."
```

### 5. List open pull requests

```bash
gh pr list --state open
```

### 6. View PR details and checks

Replace `PR_NUMBER` with the actual pull request number:

```bash
gh pr view PR_NUMBER
gh pr view PR_NUMBER --json reviewRequests,reviews,reviewDecision
gh pr checks PR_NUMBER
```

To monitor checks until completion:

```bash
gh pr checks PR_NUMBER --watch
```

### 7. Merge the pull request

Review the PR and check its status before merging.

```bash
gh pr view PR_NUMBER
gh pr checks PR_NUMBER
```

Merge using squash, if enabled by repository settings:

```bash
gh pr merge PR_NUMBER --squash
```

Update the local base branch afterward:

```bash
git switch master
git pull origin master
```

### Merge methods supported by `gh pr merge`

- `--merge`: Create a merge commit.
- `--squash`: Combine the PR commits into one commit.
- `--rebase`: Replay the PR commits onto the base branch without creating a merge commit.

The repository settings and branch protection rules determine which methods are allowed.

### How to review someone else's PR

```bash
gh pr list --repo OWNER/REPO
gh pr view PR_NUMBER --repo OWNER/REPO
gh pr diff PR_NUMBER --repo OWNER/REPO
gh pr checks PR_NUMBER --repo OWNER/REPO
```

Submit a review after inspecting the code:

```bash
gh pr review PR_NUMBER --approve \
  --body "Reviewed the changes. Looks good."
```

Other review options include `--comment` for feedback and `--request-changes` for changes that must be addressed.

### Observation

Pull requests provide a controlled way to review code before merging. GitHub CLI allows developers to create branches, submit PRs, inspect checks, and merge changes from the terminal.

---

## Task 5: GitHub Actions and Workflows

For this preview, I used the public repository `cli/cli`.

### 1. List workflows

```bash
gh workflow list --repo cli/cli
```

### 2. List workflow runs

```bash
gh run list --repo cli/cli --limit 10
```

### 3. View a specific workflow run

Replace `RUN_ID` with an actual run ID returned by the previous command:

```bash
gh run view RUN_ID --repo cli/cli
```

### 4. Inspect failed logs

```bash
gh run view RUN_ID --repo cli/cli --log-failed
```

### 5. Monitor a workflow

```bash
gh run watch RUN_ID --repo cli/cli
```

### How are `gh run` and `gh workflow` useful in CI/CD?

- `gh run list` helps identify failed or successful pipeline runs.
- `gh run view` helps investigate build and test failures.
- `gh run watch` monitors a workflow until completion.
- `gh run rerun` retries failed jobs.
- `gh workflow list` identifies available workflows.
- `gh workflow view` helps inspect workflow configuration.
- `gh workflow run` can manually trigger workflows that support `workflow_dispatch`.

### Observation

These commands help DevOps engineers monitor CI/CD pipelines, diagnose failures, and trigger supported automation without repeatedly opening the GitHub website.

---

## Task 6: Useful GitHub CLI Tricks

### 1. GitHub API

```bash
gh api user --jq '.login'
gh api repos/OWNER/REPO --jq '.stargazers_count'
```

**Use:** Retrieve account information and repository metadata from the GitHub API.

### 2. GitHub Gists

```bash
gh gist list
gh gist create notes.txt --desc "DevOps notes"
```

**Use:** Share code snippets, scripts, and short technical notes.

### 3. GitHub Releases

```bash
gh release list
gh release view v1.0.0
gh release create v1.0.0 --generate-notes
```

**Use:** Publish versioned software releases and generate release notes.

### 4. GitHub CLI Aliases

```bash
gh alias set co 'pr checkout'
gh alias list
```

Example:

```bash
gh co 123
```

**Use:** Create shortcuts for commands used frequently.

### 5. Search repositories

```bash
gh search repos "devops" --limit 10
gh search repos "kubernetes" --language go --limit 10
```

**Use:** Discover repositories and explore open-source projects from the terminal.

## 7. GitHub CLI Commands and Their Uses

### Task 1: Installation and Authentication

| Command | Use |
|---|---|
| `sudo apt update` | Updates the Ubuntu package index. |
| `sudo apt install -y gh` | Installs GitHub CLI. |
| `gh --version` | Displays the installed CLI version. |
| `gh auth login` | Authenticates your GitHub account. |
| `gh auth status` | Checks your authentication status. |
| `gh auth logout` | Signs out of GitHub CLI. |
| `gh auth refresh -h github.com -s delete_repo` | Requests permission to delete repositories. |
| `gh api user --jq '.login'` | Displays your authenticated GitHub username. |

### Task 2: Repository Management

| Command | Use |
|---|---|
| `gh repo create` | Creates a new GitHub repository. |
| `gh repo create gh-cli-practice --public --add-readme --clone` | Creates a public repository, adds a README, and clones it. |
| `gh repo clone OWNER/REPO` | Clones a repository to your local machine. |
| `gh repo list USERNAME --limit 100` | Lists up to 100 repositories owned by a user. |
| `gh repo view` | Displays details of the current repository. |
| `gh repo view --json url --jq '.url'` | Prints the repository URL. |
| `gh repo delete OWNER/REPO` | Deletes a repository after confirmation. |
| `gh browse --no-browser` | Displays the relevant GitHub URL without opening a browser. |

### Task 3: Issue Management

| Command | Use |
|---|---|
| `gh issue create` | Creates a new issue interactively. |
| `gh issue create --title "Bug report" --body "Describe the bug"` | Creates an issue with a title and description. |
| `gh issue list` | Lists issues in the current repository. |
| `gh issue list --state open` | Lists only open issues. |
| `gh issue view 1` | Displays issue number 1. |
| `gh issue close 1` | Closes issue number 1. |
| `gh issue reopen 1` | Reopens issue number 1. |
| `gh issue create --label enhancement` | Creates an issue with the `enhancement` label. |

### Task 4: Pull Request Management

| Command | Use |
|---|---|
| `git checkout -b feature-branch` | Creates and switches to a new branch. |
| `git add .` | Stages all changes in the current directory. |
| `git commit -m "Add documentation"` | Commits staged changes. |
| `git push -u origin feature-branch` | Pushes the branch to GitHub and sets its upstream. |
| `gh pr create` | Creates a pull request interactively. |
| `gh pr create --base master --head feature-branch` | Creates a pull request targeting `master`. |
| `gh pr list` | Lists pull requests in the current repository. |
| `gh pr view 1` | Displays pull request number 1. |
| `gh pr diff 1` | Shows the changes in a pull request. |
| `gh pr checks 1` | Displays CI check results for a pull request. |
| `gh pr review 1 --approve` | Approves a pull request. |
| `gh pr review 1 --comment --body "Looks good"` | Adds a review comment. |
| `gh pr merge 1 --merge` | Merges a pull request using a merge commit. |
| `gh pr merge 1 --squash` | Combines the pull request's commits into one commit. |
| `gh pr merge 1 --rebase` | Merges a pull request using rebase. |

### Task 5: GitHub Actions and Workflows

| Command | Use |
|---|---|
| `gh workflow list` | Lists workflows in the current repository. |
| `gh workflow view WORKFLOW` | Displays workflow details. |
| `gh workflow run WORKFLOW` | Manually triggers a workflow that supports `workflow_dispatch`. |
| `gh run list` | Lists recent workflow runs. |
| `gh run view RUN_ID` | Displays a workflow run's details and status. |
| `gh run view RUN_ID --log` | Displays workflow logs. |
| `gh run view RUN_ID --log-failed` | Displays logs for failed steps. |
| `gh run watch RUN_ID` | Monitors a workflow run until it finishes. |
| `gh run rerun RUN_ID` | Reruns a workflow run. |
| `gh run cancel RUN_ID` | Cancels a running workflow. |

### Task 6: Useful Advanced Commands

| Command | Use |
|---|---|
| `gh release list` | Lists repository releases. |
| `gh release view TAG` | Displays details of a release. |
| `gh gist list` | Lists your GitHub Gists. |
| `gh gist create FILE` | Creates a Gist from a local file. |
| `gh search repos "devops"` | Searches repositories for `devops`. |
| `gh search issues "bug"` | Searches issues and pull requests for `bug`. |
| `gh secret list` | Lists repository secrets you can access. |
| `gh variable list` | Lists repository-level Actions variables. |
| `gh alias list` | Lists configured CLI aliases. |
| `gh help` | Displays general GitHub CLI help. |
| `gh help issue` | Displays help for issue commands. |
| `gh help pr` | Displays help for pull request commands. |

### Additional Linux and Git Commands

| Command | Use |
|---|---|
| `pwd` | Displays the current working directory. |
| `ls -la` | Lists all files, including hidden files. |
| `cat filename.md` | Displays a file's contents. |
| `git status` | Shows the working tree and staging status. |
| `git branch` | Lists local branches. |
| `git checkout master` | Switches to the `master` branch. |
| `git pull` | Fetches and integrates changes from the upstream branch. |
| `git log --oneline` | Displays a compact commit history. |
| `git remote -v` | Displays configured remote repository URLs. |
| `cat >> filename.md <<'EOF'` | Appends text to a file until the closing `EOF` marker. |
| `cat > filename.md <<'EOF'` | Creates or overwrites a file with the supplied text. |

### Important Notes

- Replace placeholders such as `OWNER`, `REPO`, `USERNAME`, `WORKFLOW`, `RUN_ID`, and `TAG` with actual values.
- Replace issue and pull request numbers with the relevant IDs.
- Run repository-specific commands inside the correct repository, or use `--repo OWNER/REPO` where supported.
- The `master` branch is used in this practice repository; other repositories may use `main`.
- Review a command carefully before running destructive operations such as repository deletion.


---

## Key Learnings

1. GitHub CLI allows repository and issue management directly from the terminal.
2. Authentication scopes and repository permissions both affect what operations are allowed.
3. Branches and pull requests support structured code review and collaboration.
4. GitHub Actions can be monitored and investigated using `gh run` and `gh workflow`.
5. GitHub API access enables scripting and integration with automation tools.
6. CLI commands can be combined with Bash scripts to automate repetitive DevOps tasks.
7. On headless EC2 instances, browser URLs can be retrieved from the terminal and opened on a local computer.
