---
title: "Git reference Guide"
description: "Git & GitHub — Complete Reference Guide"
---

> 40 topics · 5 categories · Short explanations with examples · Every topic has a direct anchor link and a back-to-top link

---

<a id="table-of-contents"></a>
## Table of Contents

### 🟢 Git Core
| # | Topic | Jump |
|---|-------|------|
| 01 | `git init` — Initialize a repository | [→ Go](#git-init) |
| 02 | `git clone` — Copy a remote repository | [→ Go](#git-clone) |
| 03 | `git add` & Staging | [→ Go](#git-add) |
| 04 | `git commit` — Save a snapshot | [→ Go](#git-commit) |
| 05 | `git status` & `git diff` | [→ Go](#git-status) |
| 06 | `git log` — Browse history | [→ Go](#git-log) |
| 07 | `git branch` — Manage branches | [→ Go](#git-branch) |
| 08 | `git checkout` / `git switch` | [→ Go](#git-checkout) |
| 09 | `git merge` — Combine branches | [→ Go](#git-merge) |
| 10 | `git rebase` — Rewrite history | [→ Go](#git-rebase) |
| 11 | `git stash` — Shelve changes | [→ Go](#git-stash) |
| 12 | `git remote` — Manage remotes | [→ Go](#git-remote) |
| 13 | `git push` — Upload commits | [→ Go](#git-push) |
| 14 | `git pull` & `git fetch` | [→ Go](#git-pull) |
| 15 | `git reset` — Undo commits | [→ Go](#git-reset) |
| 16 | `git revert` — Safe undo | [→ Go](#git-revert) |
| 17 | `git tag` — Mark releases | [→ Go](#git-tag) |
| 18 | `.gitignore` — Ignore files | [→ Go](#gitignore) |
| 19 | `git config` — Configuration | [→ Go](#git-config) |

### 🔵 GitHub
| # | Topic | Jump |
|---|-------|------|
| 20 | GitHub Repositories | [→ Go](#github-repo) |
| 21 | GitHub Pages | [→ Go](#github-pages) |
| 22 | GitHub Releases | [→ Go](#github-releases) |
| 23 | GitHub CLI (`gh`) | [→ Go](#github-cli) |
| 24 | GitHub Codespaces | [→ Go](#github-codespaces) |
| 25 | GitHub Packages | [→ Go](#github-packages) |

### 🟡 Workflow
| # | Topic | Jump |
|---|-------|------|
| 26 | GitHub Actions — CI/CD | [→ Go](#github-actions) |
| 27 | Secrets & Environment Variables | [→ Go](#github-secrets) |
| 28 | Conventional Commits | [→ Go](#conventional-commits) |
| 29 | Branching Strategies | [→ Go](#git-flow) |

### 🩷 Collaboration
| # | Topic | Jump |
|---|-------|------|
| 30 | Forking | [→ Go](#github-fork) |
| 31 | Pull Requests | [→ Go](#github-pr) |
| 32 | Issues | [→ Go](#github-issues) |
| 33 | Branch Protection Rules | [→ Go](#branch-protection) |
| 34 | Code Review | [→ Go](#code-review) |
| 35 | CODEOWNERS | [→ Go](#codeowners) |

### 🟣 Advanced Git
| # | Topic | Jump |
|---|-------|------|
| 36 | `git cherry-pick` | [→ Go](#git-cherry-pick) |
| 37 | `git bisect` — Debug with binary search | [→ Go](#git-bisect) |
| 38 | `git worktree` — Multiple checkouts | [→ Go](#git-worktree) |
| 39 | `git submodule` — Nested repositories | [→ Go](#git-submodule) |
| 40 | Git Hooks — Lifecycle scripts | [→ Go](#git-hooks) |

---

<!-- ============================================================ -->
<!--                      GIT CORE                                -->
<!-- ============================================================ -->

<a id="git-init"></a>
## 01 · `git init` — Initialize a repository

**Category:** Git Core · **Anchor:** `#git-init`

Creates a new empty Git repository in the current directory by adding a hidden `.git/` folder that stores the entire version history, configuration, and object database.

```bash
# Start a brand-new project
git init my-project
cd my-project

# Or initialize Git inside an existing folder
cd existing-folder
git init

# Check what was created
ls -la .git/
# drwxr-xr-x  HEAD  config  description  hooks/  info/  objects/  refs/
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-clone"></a>
## 02 · `git clone` — Copy a remote repository

**Category:** Git Core · **Anchor:** `#git-clone`

Downloads a complete copy of a remote repository to your local machine — including all files, commits, branches, and tags. Automatically sets `origin` as the remote name.

```bash
# Clone a GitHub repository
git clone https://github.com/user/repo.git

# Clone into a custom folder name
git clone https://github.com/user/repo.git my-folder

# Clone only the latest snapshot (faster, less disk space)
git clone --depth 1 https://github.com/user/repo.git

# Clone a specific branch
git clone --branch develop https://github.com/user/repo.git
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-add"></a>
## 03 · `git add` — Staging changes

**Category:** Git Core · **Anchor:** `#git-add`

Moves changes from the working directory into the staging area (also called the index). Only staged changes are included in the next commit. This two-step process lets you craft precise commits.

```bash
# Stage a single file
git add README.md

# Stage all changes in the current directory
git add .

# Stage all changes in the whole repository
git add -A

# Interactively choose which hunks to stage (very useful)
git add -p

# Unstage a file (keep the changes, just remove from staging)
git restore --staged README.md
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-commit"></a>
## 04 · `git commit` — Save a snapshot

**Category:** Git Core · **Anchor:** `#git-commit`

Permanently records all staged changes as a new commit in the repository history. Each commit gets a unique SHA hash and stores author, date, and message.

```bash
# Commit staged changes with an inline message
git commit -m "feat: add user login page"

# Stage all tracked files and commit in one step
git commit -am "fix: correct typo in README"

# Amend the last commit (change message or add forgotten files)
git add forgotten-file.js
git commit --amend --no-edit

# Open the configured editor to write a detailed message
git commit
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-status"></a>
## 05 · `git status` & `git diff`

**Category:** Git Core · **Anchor:** `#git-status`

`git status` shows which files are untracked, modified, or staged. `git diff` shows the exact line-by-line changes — unstaged by default, or staged with `--staged`.

```bash
# Summary of current working tree state
git status

# Short format
git status -s

# Show unstaged changes (working tree vs index)
git diff

# Show staged changes (index vs last commit)
git diff --staged

# Compare two branches
git diff main..feature/login

# Show only the names of changed files
git diff --name-only HEAD~1
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-log"></a>
## 06 · `git log` — Browse commit history

**Category:** Git Core · **Anchor:** `#git-log`

Displays the commit history of the repository. Supports extensive formatting options to filter by author, date range, file, or keyword.

```bash
# Clean one-line graph of all branches
git log --oneline --graph --all

# Filter by author
git log --author="Alice" --oneline

# Filter by date
git log --since="2 weeks ago" --until="yesterday"

# Show the history of a specific file
git log -p src/auth.js

# Search commit messages
git log --grep="login" --oneline

# Show commit that introduced a specific line
git log -S "passwordHash" --oneline
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-branch"></a>
## 07 · `git branch` — Manage branches

**Category:** Git Core · **Anchor:** `#git-branch`

Branches are lightweight, movable pointers to commits. Creating a branch is instant and cheap. Use them freely to isolate features, fixes, and experiments.

```bash
# List all local branches
git branch

# List all local and remote branches
git branch -a

# Create a new branch (does not switch to it)
git branch feature/user-auth

# Delete a branch that has been merged
git branch -d feature/user-auth

# Force delete an unmerged branch
git branch -D feature/abandoned

# Rename the current branch
git branch -m new-name
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-checkout"></a>
## 08 · `git checkout` / `git switch` & `git restore`

**Category:** Git Core · **Anchor:** `#git-checkout`

Modern Git splits the old `git checkout` command into two: `git switch` for changing branches and `git restore` for discarding file changes. Both are clearer and safer.

```bash
# Switch to an existing branch (modern syntax)
git switch main

# Create and switch to a new branch
git switch -c feature/payments

# Switch back to the previous branch
git switch -

# Discard changes to a file in the working tree
git restore index.html

# Restore a file to a specific commit's version
git restore --source abc1234 src/config.js

# Legacy syntax (still works)
git checkout -b feature/payments
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-merge"></a>
## 09 · `git merge` — Combine branches

**Category:** Git Core · **Anchor:** `#git-merge`

Joins the history of two branches. If the target branch has not diverged, Git performs a fast-forward. Otherwise it creates a merge commit. Conflicts must be resolved manually.

```bash
# Switch to the receiving branch first
git switch main

# Merge a feature branch into main
git merge feature/login

# Always create a merge commit (no fast-forward)
git merge --no-ff feature/login

# Abort a merge with conflicts
git merge --abort

# After resolving conflicts, mark resolved and continue
git add conflicted-file.js
git merge --continue
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-rebase"></a>
## 10 · `git rebase` — Rewrite history

**Category:** Git Core · **Anchor:** `#git-rebase`

Moves or replays commits from one branch onto another, producing a clean linear history. Because it rewrites commit SHAs, **never rebase commits that have been pushed to a shared branch**.

```bash
# Rebase the current feature branch onto main
git switch feature/login
git rebase main

# Interactive rebase: squash, reorder, or edit last 3 commits
git rebase -i HEAD~3

# In the interactive editor, change 'pick' to:
# squash   → combine with previous commit
# reword   → edit the commit message
# drop     → remove the commit entirely

# Abort a rebase in progress
git rebase --abort
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-stash"></a>
## 11 · `git stash` — Shelve changes temporarily

**Category:** Git Core · **Anchor:** `#git-stash`

Saves your uncommitted work to a stack so you can switch context — pull changes, switch branches, or apply an urgent fix — then come back and re-apply your in-progress work.

```bash
# Stash all uncommitted changes
git stash

# Stash with a descriptive name
git stash push -m "WIP: refactor auth middleware"

# List all stashes
git stash list

# Re-apply the most recent stash and remove it from the stack
git stash pop

# Apply a specific stash without removing it
git stash apply stash@{2}

# Drop a specific stash
git stash drop stash@{0}
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-remote"></a>
## 12 · `git remote` — Manage remote connections

**Category:** Git Core · **Anchor:** `#git-remote`

Remotes are named references to other repositories (usually hosted on GitHub). `origin` is the conventional name for the primary remote. `upstream` is used for the original repo when you work with forks.

```bash
# List all remotes with URLs
git remote -v

# Add a new remote
git remote add origin https://github.com/user/repo.git

# Add upstream for fork workflow
git remote add upstream https://github.com/original/repo.git

# Remove a remote
git remote remove backup

# Rename a remote
git remote rename origin old-origin

# Update a remote URL
git remote set-url origin https://github.com/user/new-repo.git
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-push"></a>
## 13 · `git push` — Upload commits

**Category:** Git Core · **Anchor:** `#git-push`

Sends local commits to the remote repository. The first push of a new branch needs `-u` to set the upstream tracking reference so future pushes can use the short form `git push`.

```bash
# Push current branch to origin
git push origin main

# Push and set upstream tracking (first push of a branch)
git push -u origin feature/payments

# After -u is set, just use:
git push

# Push all local tags
git push origin --tags

# Safer force push (only overwrites if remote has not changed)
git push --force-with-lease

# Delete a remote branch
git push origin --delete feature/old-feature
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-pull"></a>
## 14 · `git pull` & `git fetch`

**Category:** Git Core · **Anchor:** `#git-pull`

`git fetch` downloads remote changes without touching your working tree. `git pull` is `git fetch` + `git merge` in one step. Using `--rebase` instead of merge keeps history linear.

```bash
# Download changes without merging (safe to inspect first)
git fetch origin

# See what was fetched before merging
git log HEAD..origin/main --oneline

# Pull (fetch + merge)
git pull

# Pull with rebase instead of merge (cleaner history)
git pull --rebase

# Pull a specific branch from a specific remote
git pull upstream main
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-reset"></a>
## 15 · `git reset` — Undo commits

**Category:** Git Core · **Anchor:** `#git-reset`

Moves the HEAD pointer (and optionally the index and working tree) backwards in history. Three modes: `--soft` keeps everything staged, `--mixed` (default) unstages changes, `--hard` discards all changes permanently.

```bash
# Undo last commit — keep changes staged
git reset --soft HEAD~1

# Undo last commit — keep changes in working tree (default)
git reset HEAD~1

# Undo last commit — DISCARD all changes (destructive!)
git reset --hard HEAD~1

# Unstage a specific file (does not delete changes)
git reset HEAD src/app.js

# Reset to a specific commit (dangerous on shared branches)
git reset --hard abc1234
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-revert"></a>
## 16 · `git revert` — Safe undo

**Category:** Git Core · **Anchor:** `#git-revert`

Creates a new commit that reverses the changes of a previous commit. Unlike `git reset`, it does not rewrite history, making it safe to use on shared branches that others have already pulled.

```bash
# Revert the last commit (creates a new "undo" commit)
git revert HEAD

# Revert a specific commit by hash
git revert abc1234

# Revert without opening the editor (accept default message)
git revert abc1234 --no-edit

# Revert a range of commits
git revert HEAD~3..HEAD

# Stage the revert but do not commit yet
git revert -n abc1234
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-tag"></a>
## 17 · `git tag` — Mark releases

**Category:** Git Core · **Anchor:** `#git-tag`

Tags are named references to specific commits, typically used to mark release versions. Annotated tags store extra metadata (tagger name, date, message) and are recommended for releases.

```bash
# Create a lightweight tag on the current commit
git tag v1.0.0

# Create an annotated tag (recommended for releases)
git tag -a v1.0.0 -m "First stable release"

# Tag a specific past commit
git tag -a v0.9.0 abc1234 -m "Beta release"

# List all tags
git tag -l

# Push a single tag to remote
git push origin v1.0.0

# Push all tags to remote
git push origin --tags

# Delete a local tag
git tag -d v1.0.0-beta
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="gitignore"></a>
## 18 · `.gitignore` — Ignore files

**Category:** Git Core · **Anchor:** `#gitignore`

A plain text file listing patterns for files and directories that Git should never track. Patterns support wildcards. A global `~/.gitignore` applies to every repository on your machine.

```bash
# Example .gitignore
node_modules/          # dependency folder
dist/                  # build output
.env                   # local environment secrets
*.log                  # all log files
.DS_Store              # macOS metadata files
*.pyc                  # Python bytecode
.idea/                 # JetBrains IDE files
coverage/              # test coverage reports

# Check if a specific file is being ignored (and why)
git check-ignore -v path/to/file.log

# Set up a global gitignore for all repos
git config --global core.excludesFile ~/.gitignore_global
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-config"></a>
## 19 · `git config` — Configuration

**Category:** Git Core · **Anchor:** `#git-config`

Sets Git preferences at three levels: `--local` (this repo only), `--global` (your user account), or `--system` (all users on the machine). The most specific level wins.

```bash
# Set your identity (required for commits)
git config --global user.name "Alice Smith"
git config --global user.email "alice@example.com"

# Set VS Code as the default editor
git config --global core.editor "code --wait"

# Set rebase as the default pull strategy
git config --global pull.rebase true

# Create a short alias
git config --global alias.lg "log --oneline --graph --all"
git lg   # now works as a shortcut

# View all global settings
git config --list --global
```

[↑ Back to Table of Contents](#table-of-contents)

---

<!-- ============================================================ -->
<!--                        GITHUB                                -->
<!-- ============================================================ -->

<a id="github-repo"></a>
## 20 · GitHub Repositories

**Category:** GitHub · **Anchor:** `#github-repo`

A GitHub repository is the central hub for your project — it hosts source code, issues, pull requests, wikis, Actions workflows, and security settings. Repositories can be public, private, or internal (for organisations).

```bash
# Create a new public repo via GitHub CLI
gh repo create my-project --public --clone

# Create a private repo with a description
gh repo create my-api --private --description "REST API service"

# View a repository's details
gh repo view user/my-project

# List your repositories
gh repo list --limit 20

# Archive a repository (makes it read-only)
gh repo archive user/old-project
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-pages"></a>
## 21 · GitHub Pages

**Category:** GitHub · **Anchor:** `#github-pages`

Host static websites directly from a GitHub repository at no cost. Supports custom domains, HTTPS, and Jekyll. Great for documentation, portfolios, and project sites.

```bash
# Deploy from the /docs folder on main branch (via API)
gh api repos/:owner/:repo/pages \
  --method POST \
  --field source[branch]=main \
  --field source[path]=/docs

# Or deploy using the gh-pages branch convention
git switch --orphan gh-pages
git commit --allow-empty -m "init gh-pages"
git push origin gh-pages

# Use the actions/deploy-pages action for full control:
# - uses: actions/deploy-pages@v3
#   with:
#     artifact_name: github-pages
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-releases"></a>
## 22 · GitHub Releases

**Category:** GitHub · **Anchor:** `#github-releases`

Package tagged versions with human-readable release notes and optional binary assets (executables, archives). GitHub can auto-generate changelogs from merged pull request titles.

```bash
# Create a release from an existing tag
gh release create v1.2.0 \
  --title "v1.2.0 — Performance update" \
  --notes "Reduces API response time by 40%. Fixes #98, #101."

# Attach binary assets to a release
gh release create v1.2.0 \
  --title "v1.2.0" \
  --generate-notes \
  ./dist/app-linux-amd64.tar.gz \
  ./dist/app-darwin-arm64.tar.gz

# List recent releases
gh release list

# Download assets from a release
gh release download v1.2.0 --pattern "*.tar.gz"
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-cli"></a>
## 23 · GitHub CLI (`gh`)

**Category:** GitHub · **Anchor:** `#github-cli`

The official command-line tool for GitHub. Manage repositories, pull requests, issues, Actions, Codespaces, and Gists without leaving the terminal.

```bash
# Authenticate with your GitHub account
gh auth login

# List open pull requests
gh pr list --state open

# Create an issue
gh issue create --title "Bug: 500 on checkout" --label "bug"

# Run a workflow manually
gh workflow run deploy.yml --field environment=staging

# Create a Gist from a file
gh gist create snippet.js --public --desc "Utility functions"

# Open the current repo in the browser
gh repo view --web
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-codespaces"></a>
## 24 · GitHub Codespaces

**Category:** GitHub · **Anchor:** `#github-codespaces`

Fully configured, cloud-hosted development environments that launch in seconds from any repository. Accessible via a browser or VS Code. Configuration lives in `.devcontainer/devcontainer.json`.

```bash
# Create a codespace for a repo
gh codespace create --repo user/my-project --branch main

# List all your codespaces
gh codespace list

# Connect to a codespace via SSH
gh codespace ssh

# Forward a port from a codespace to localhost
gh codespace ports forward 3000:3000

# Stop a running codespace
gh codespace stop --codespace my-project-abc123

# Open in browser
gh codespace view --web
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-packages"></a>
## 25 · GitHub Packages

**Category:** GitHub · **Anchor:** `#github-packages`

Host and share packages (npm, Docker, Maven, NuGet, RubyGems) privately or publicly alongside your source code. Integrates with GitHub Actions for automated publishing.

```bash
# Publish an npm package to GitHub Packages
# 1. Set registry in .npmrc:
# @myorg:registry=https://npm.pkg.github.com

# 2. Authenticate
npm login --registry=https://npm.pkg.github.com

# 3. Publish
npm publish

# Pull a Docker image from GitHub Container Registry
docker pull ghcr.io/user/my-image:latest

# Push a Docker image
docker tag my-image ghcr.io/user/my-image:v1.0.0
docker push ghcr.io/user/my-image:v1.0.0
```

[↑ Back to Table of Contents](#table-of-contents)

---

<!-- ============================================================ -->
<!--                        WORKFLOW                              -->
<!-- ============================================================ -->

<a id="github-actions"></a>
## 26 · GitHub Actions — CI/CD

**Category:** Workflow · **Anchor:** `#github-actions`

Automate build, test, and deploy workflows triggered by events: `push`, `pull_request`, `schedule`, `workflow_dispatch`, and more. Workflows are YAML files stored in `.github/workflows/`.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-secrets"></a>
## 27 · Secrets & Environment Variables

**Category:** Workflow · **Anchor:** `#github-secrets`

Store sensitive values (API keys, passwords, tokens) as encrypted secrets in GitHub. Variables are for non-sensitive config values. Both are referenced in workflows via `${{ secrets.NAME }}` and `${{ vars.NAME }}`.

```bash
# Set a secret via GitHub CLI (prompts for value)
gh secret set AWS_ACCESS_KEY_ID

# Set a secret from a file
gh secret set TLS_CERT < ./cert.pem

# Set a repository variable (non-sensitive)
gh variable set NODE_ENV --body "production"

# List all secrets (names only — values are never shown)
gh secret list
```

```yaml
# Reference in a workflow:
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  NODE_ENV: ${{ vars.NODE_ENV }}
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="conventional-commits"></a>
## 28 · Conventional Commits

**Category:** Workflow · **Anchor:** `#conventional-commits`

A specification for structuring commit messages: `type(scope): description`. Enables automated changelog generation, semantic versioning, and better `git log` readability. Supported by tools like `commitlint`, `semantic-release`, and `standard-version`.

```bash
# Format: type(optional-scope): short description
#
# Types:
# feat     — a new feature (triggers minor version bump)
# fix      — a bug fix (triggers patch version bump)
# docs     — documentation only changes
# style    — formatting, whitespace (no logic change)
# refactor — code change with no bug fix or feature
# test     — adding or updating tests
# chore    — build process, dependency updates
# perf     — performance improvement
# ci       — CI/CD configuration changes

git commit -m "feat(auth): add Google OAuth2 login"
git commit -m "fix(api): handle null response from /users endpoint"
git commit -m "docs: update README with Docker setup instructions"
git commit -m "chore!: drop support for Node 14"
#                    ^ breaking change (triggers major version bump)
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-flow"></a>
## 29 · Branching Strategies

**Category:** Workflow · **Anchor:** `#git-flow`

Three popular models: **Git Flow** (feature/develop/main/hotfix branches — complex), **GitHub Flow** (feature branches + PRs into main — simple), and **Trunk-Based Development** (commit directly to main with feature flags — fastest CI).

```bash
# GitHub Flow (recommended for most teams):
# 1. Create a branch from main
git switch -c feature/add-search

# 2. Make commits
git commit -m "feat: implement full-text search"

# 3. Push and open a PR
git push -u origin feature/add-search
gh pr create --title "Add full-text search" --base main

# 4. After review, merge and delete the branch
gh pr merge --squash --delete-branch

# Trunk-Based Development:
# Commit small changes directly to main with feature flags
git switch main
git commit -m "feat: add search (behind ENABLE_SEARCH flag)"
git push
```

[↑ Back to Table of Contents](#table-of-contents)

---

<!-- ============================================================ -->
<!--                      COLLABORATION                           -->
<!-- ============================================================ -->

<a id="github-fork"></a>
## 30 · Forking

**Category:** Collaboration · **Anchor:** `#github-fork`

Creates a personal copy of someone else's repository on your GitHub account. You can freely experiment, then contribute changes back via a pull request. The fork stays connected to the original (upstream) repository.

```bash
# Fork a repo via GitHub CLI
gh repo fork user/upstream-repo

# Clone your fork locally
git clone https://github.com/your-username/upstream-repo.git

# Add the original repo as upstream remote
git remote add upstream https://github.com/user/upstream-repo.git

# Sync your fork with upstream
git fetch upstream
git switch main
git merge upstream/main

# Push the synced main to your fork
git push origin main
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-pr"></a>
## 31 · Pull Requests

**Category:** Collaboration · **Anchor:** `#github-pr`

A pull request (PR) proposes that your branch be merged into another. It enables code review, inline comments, CI status checks, and discussion before changes land in the main branch.

```bash
# Create a pull request
gh pr create \
  --title "feat: add dark mode support" \
  --body "Closes #55. Adds a theme toggle in user settings." \
  --base main \
  --label "enhancement"

# Check out a PR locally to test it
gh pr checkout 123

# View a PR's status and checks
gh pr view 123

# List all open PRs
gh pr list --state open

# Merge a PR (squash + auto-delete branch)
gh pr merge 123 --squash --delete-branch
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="github-issues"></a>
## 32 · Issues

**Category:** Collaboration · **Anchor:** `#github-issues`

Track bugs, feature requests, tasks, and questions. Issues support labels, milestones, assignees, and linking to commits and PRs. Closing keywords in commits/PRs automatically close linked issues.

```bash
# Create an issue
gh issue create \
  --title "Bug: 500 error on checkout page" \
  --body "Steps to reproduce: 1. Add item to cart 2. Click checkout" \
  --label "bug" \
  --assignee "@me"

# List all open bugs
gh issue list --label "bug" --state open

# Close an issue manually
gh issue close 42

# Auto-close via a commit message (closing keywords):
git commit -m "fix: resolve cart total calculation

Fixes #42
Closes #43"
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="branch-protection"></a>
## 33 · Branch Protection Rules

**Category:** Collaboration · **Anchor:** `#branch-protection`

Enforce quality gates on important branches like `main`. Require pull request reviews, passing CI status checks, up-to-date branches, signed commits, or linear history before a merge is allowed.

```bash
# Set branch protection via GitHub REST API (using gh api)
gh api repos/:owner/:repo/branches/main/protection \
  --method PUT \
  --field required_status_checks[strict]=true \
  --field required_status_checks[contexts][]=ci/test \
  --field enforce_admins=true \
  --field required_pull_request_reviews[required_approving_review_count]=1 \
  --field required_pull_request_reviews[dismiss_stale_reviews]=true \
  --field allow_force_pushes=false \
  --field allow_deletions=false
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="code-review"></a>
## 34 · Code Review

**Category:** Collaboration · **Anchor:** `#code-review`

The process of systematically examining code changes in a PR before merging. Good reviews catch bugs, enforce style, share knowledge, and improve overall quality. GitHub supports inline comments and suggested code changes.

```bash
# Check out a PR to test it locally
gh pr checkout 123

# Leave a comment on a PR
gh pr review 123 \
  --comment \
  --body "The database query on line 45 is missing an index — could cause slow performance at scale."

# Approve a PR
gh pr review 123 --approve

# Request changes on a PR
gh pr review 123 \
  --request-changes \
  --body "Please add unit tests for the new auth flow before merging."
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="codeowners"></a>
## 35 · CODEOWNERS

**Category:** Collaboration · **Anchor:** `#codeowners`

A `.github/CODEOWNERS` file maps file paths to GitHub users or teams. When a PR touches an owned file, GitHub automatically requests a review from the defined owner. Branch protection can require CODEOWNER approval.

```bash
# .github/CODEOWNERS

# Default owner for everything in the repo
*                          @org/core-team

# Frontend is owned by the frontend team
/src/frontend/             @org/frontend-team

# A specific file owned by an individual
/src/payments/stripe.js    @alice

# All Markdown documentation owned by the docs team
*.md                       @org/docs-team

# Infrastructure files owned by DevOps
/infra/                    @bob @carol
/.github/workflows/        @org/devops-team
```

[↑ Back to Table of Contents](#table-of-contents)

---

<!-- ============================================================ -->
<!--                      ADVANCED GIT                            -->
<!-- ============================================================ -->

<a id="git-cherry-pick"></a>
## 36 · `git cherry-pick`

**Category:** Advanced Git · **Anchor:** `#git-cherry-pick`

Applies a specific commit from any branch onto your current branch without merging the entire branch. Useful for backporting fixes to older release branches.

```bash
# Apply a single commit from another branch
git cherry-pick abc1234

# Apply a range of commits
git cherry-pick abc1234..def5678

# Cherry-pick without immediately committing (stage only)
git cherry-pick -n abc1234

# Cherry-pick onto a release branch (common backport pattern)
git switch release/v1.2
git cherry-pick abc1234   # apply the bug fix from main
git push origin release/v1.2
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-bisect"></a>
## 37 · `git bisect` — Debug with binary search

**Category:** Advanced Git · **Anchor:** `#git-bisect`

Automates the process of finding the exact commit that introduced a bug by performing a binary search through commit history. Git checks out commits halfway between good and bad until the culprit is isolated.

```bash
# Start a bisect session
git bisect start

# Mark the current commit as bad (has the bug)
git bisect bad HEAD

# Mark a known-good commit (before the bug existed)
git bisect good v1.0.0

# Git checks out the midpoint commit.
# Test it, then tell Git the result:
git bisect good    # if this commit is fine
git bisect bad     # if this commit has the bug

# Repeat until Git identifies the first bad commit.
# When done, always reset to restore your branch:
git bisect reset
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-worktree"></a>
## 38 · `git worktree` — Multiple checkouts

**Category:** Advanced Git · **Anchor:** `#git-worktree`

Allows you to check out multiple branches simultaneously into separate directories from a single repository clone. Useful for reviewing a PR while continuing work on your own feature without stashing.

```bash
# Add a worktree for a hotfix branch in a sibling directory
git worktree add ../hotfix-critical hotfix/payment-null-crash

# Work in the new directory
cd ../hotfix-critical
# ...make fixes, commit, push...

# List all active worktrees
git worktree list

# Remove the worktree when done
git worktree remove ../hotfix-critical

# Prune stale worktree references
git worktree prune
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-submodule"></a>
## 39 · `git submodule` — Nested repositories

**Category:** Advanced Git · **Anchor:** `#git-submodule`

Embeds one Git repository inside another as a fixed reference to a specific commit. Used for shared libraries, vendor code, or design systems that are developed independently.

```bash
# Add a submodule
git submodule add https://github.com/org/shared-lib.git libs/shared-lib

# Initialize and clone submodules after a fresh clone
git submodule update --init --recursive

# Clone a repo and its submodules in one command
git clone --recurse-submodules https://github.com/user/repo.git

# Update all submodules to their latest remote commit
git submodule update --remote

# Remove a submodule
git submodule deinit libs/shared-lib
git rm libs/shared-lib
rm -rf .git/modules/libs/shared-lib
```

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="git-hooks"></a>
## 40 · Git Hooks — Lifecycle scripts

**Category:** Advanced Git · **Anchor:** `#git-hooks`

Scripts that Git runs automatically at specific lifecycle events. Client-side hooks live in `.git/hooks/` and are not versioned. Use **Husky** to share hooks with your team via `package.json`.

```bash
# Create a pre-commit hook that runs linting
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
npm run lint
if [ $? -ne 0 ]; then
  echo "Linting failed. Commit aborted."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-commit

# Create a commit-msg hook that enforces Conventional Commits
cat > .git/hooks/commit-msg << 'EOF'
#!/bin/sh
npx --no -- commitlint --edit "$1"
EOF
chmod +x .git/hooks/commit-msg

# Team-shared hooks with Husky (recommended):
npm install --save-dev husky
npx husky init
# Edit .husky/pre-commit to add your commands
```

[↑ Back to Table of Contents](#table-of-contents)

---

## Quick Reference Cheat Sheet

| Task | Command |
|---|---|
| Initialize repo | `git init` |
| Clone a repo | `git clone <url>` |
| Stage all changes | `git add .` |
| Commit staged changes | `git commit -m "message"` |
| Push to remote | `git push origin main` |
| Pull latest changes | `git pull --rebase` |
| Create + switch branch | `git switch -c feature/x` |
| Merge branch into current | `git merge feature/x` |
| Rebase onto main | `git rebase main` |
| Stash uncommitted work | `git stash` / `git stash pop` |
| Undo last commit (keep changes) | `git reset HEAD~1` |
| Safe undo on shared branch | `git revert HEAD` |
| Find bug-introducing commit | `git bisect start` |
| Apply one commit from another branch | `git cherry-pick <sha>` |
| Create a release tag | `git tag -a v1.0.0 -m "Release"` |
| Create a PR | `gh pr create` |
| Check out a PR locally | `gh pr checkout <number>` |
| Run a GitHub Actions workflow | `gh workflow run <workflow>` |

[↑ Back to Table of Contents](#table-of-contents)

---

*Git & GitHub Reference · 40 topics · All anchor links use explicit `<a id>` HTML tags — compatible with GitHub, GitLab, Bitbucket, and all standard Markdown renderers*
