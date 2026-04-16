---
title: "Git Scenario-Based Interview Questions and Answers"
description: "22 real-world Git scenario-based interview questions and answers covering branching strategies, merge/rebase, undoing changes, conflict resolution, Git internals, hooks, submodules, security, monorepos, and troubleshooting — Senior DevOps Engineer Edition."
date: 2026-04-16T14:26:30+05:30
author: "DB"
tags: ["Git", "DevOps", "CI/CD", "Interview", "Version Control", "GitHub", "GitLab"]
tool: "git"
level: "All Levels"
question_count: 22
draft: false
---

<div class="qa-list">

## 📋 Table of Contents

1. [Branching Strategies](#1-branching-strategies)
2. [Merge, Rebase & Cherry-Pick](#2-merge-rebase--cherry-pick)
3. [Undoing Changes & Recovery](#3-undoing-changes--recovery)
4. [Conflict Resolution](#4-conflict-resolution)
5. [Git Internals & Architecture](#5-git-internals--architecture)
6. [Tagging & Releases](#6-tagging--releases)
7. [Stash, Worktree & Advanced Commands](#7-stash-worktree--advanced-commands)
8. [Hooks & Automation](#8-hooks--automation)
9. [Submodules & Subtrees](#9-submodules--subtrees)
10. [Large Repos & Performance](#10-large-repos--performance)
11. [GitHub / GitLab Workflows](#11-github--gitlab-workflows)
12. [Security & Secret Management](#12-security--secret-management)
13. [Monorepo Strategies](#13-monorepo-strategies)
14. [Troubleshooting Real Incidents](#14-troubleshooting-real-incidents)

---


## 1. Branching Strategies

{{< qa num="1" q="Your team of 20 engineers is working on a SaaS product with weekly releases and hotfix needs. Which branching strategy would you recommend and why? Draw the branch flow." level="basic" >}}


**Answer:**

**Recommended: GitFlow** (for scheduled releases with hotfix support)

```
main (production)
│
│         ← hotfix/payment-bug-2.1.1
│        /                            \
│───────●──────────────────────────────●──── (tag: v2.1.1)
│      v2.1.0                          merge back to main & develop
│
develop
│
│──────●──────●──────●──────────────────────────────────────►
│     /        \      \
│    /          \      feature/user-auth
│   /            \
│  feature/       release/2.2.0
│  payment         │
│                  ├── QA, bug fixes only
│                  ├── bump version
│                  └── merge → main (tag v2.2.0) + develop
```

**Branch Rules:**

| Branch | Purpose | Who merges | Direct push? |
|--------|---------|-----------|-------------|
| `main` | Production-ready code | Release manager only | ❌ Never |
| `develop` | Integration branch | Tech leads via PR | ❌ Never |
| `feature/*` | New features | Developer → PR to develop | ✅ Own branch |
| `release/*` | Release stabilization | QA + lead | ❌ PR only |
| `hotfix/*` | Emergency prod fixes | Lead → main + develop | ❌ PR only |

**Real-world Setup:**
```bash
# Initialize GitFlow
git flow init -d

# Start a feature
git flow feature start user-authentication
# ... work ...
git flow feature finish user-authentication
# → merges to develop, deletes feature branch

# Start a release
git flow release start 2.2.0
# ... QA, version bump ...
git flow release finish 2.2.0
# → merges to main + develop, creates tag v2.2.0

# Emergency hotfix
git flow hotfix start payment-null-pointer
# ... fix ...
git flow hotfix finish payment-null-pointer
# → merges to main + develop, creates tag v2.1.1
```

**When NOT to use GitFlow:**
- Continuous deployment teams (use GitHub Flow or trunk-based instead)
- Small teams with daily releases
- Microservices with independent release cycles

{{< /qa >}}


{{< qa num="2" q="Your team wants to move from GitFlow to Trunk-Based Development. What challenges will you face and how do you manage them?" level="advanced" >}}

**Answer:**

**Trunk-Based Development (TBD) — Core Concept:**
```
main (trunk) — always deployable
│
●────●────●────●────●────●────►   (multiple commits per day per engineer)
│    │    │    │    │
│    │    │    │    └── small commit: fix login button color
│    │    │    └─────── small commit: add unit test
│    │    └──────────── small commit: refactor auth service
│    └───────────────── short-lived branch (< 2 days): feature/oauth
└────────────────────── short-lived branch (< 2 days): feature/dark-mode
```

**Key Challenges & Solutions:**

**Challenge 1: Incomplete features going to production**
```bash
# Solution: Feature Flags (LaunchDarkly, Unleash, or simple env vars)
# Code is merged but hidden behind a flag

# Example: React feature flag
const PaymentV2 = () => {
  const { isEnabled } = useFeatureFlag('payment-v2');
  return isEnabled ? <PaymentV2Component /> : <PaymentV1Component />;
};
```

**Challenge 2: Breaking other developers**
```bash
# Solution: Robust pre-merge CI — block merge if tests fail
# .github/branch-protection-rules:
# - Require status checks: unit-tests, integration-tests, lint
# - Require 1 approving review
# - Dismiss stale approvals on new push
# - Require branches to be up to date before merging

# Developer workflow:
git checkout -b feature/add-oauth    # Short-lived branch
# ... make small, focused commits ...
git push origin feature/add-oauth
# → CI runs → PR review → squash merge to main
```

**Challenge 3: Release management without release branches**
```bash
# Solution: Tags + release from main at any point
git tag -a v2.3.0 -m "Release 2.3.0 — OAuth + Dark Mode"
git push origin v2.3.0

# CI/CD pipeline triggered by tag:
# on: push: tags: ['v*']
```

**Challenge 4: Hotfixes**
```bash
# In TBD, hotfix = commit directly to main (or short branch from main)
git checkout -b hotfix/payment-crash main
git cherry-pick <fix-commit>
git push origin hotfix/payment-crash
# PR → merge to main → tag patch release
```

**Comparison:**

| Aspect | GitFlow | Trunk-Based |
|--------|---------|------------|
| Release cadence | Weekly/sprint | Continuous |
| Branch lifetime | Days to weeks | Hours to 2 days |
| CI complexity | Medium | High (robust tests needed) |
| Feature isolation | Branch isolation | Feature flags |
| Best for | Enterprise SaaS | SaaS + startups |

{{< /qa >}}


## 2. Merge, Rebase & Cherry-Pick


{{< qa num="3" q="A junior developer asks: When should I use *git merge* vs *git rebase* Give a real-world explanation with diagrams." level="advanced" >}}


**Answer:**

**`git merge` — Preserves history, creates merge commit**
```
Before merge:
main:    A──B──C
              \
feature:       D──E──F

After: git checkout main && git merge feature
main:  A──B──C──────────G  ← merge commit
              \         /
               D──E──F
```

**`git rebase` — Rewrites history, linear timeline**
```
Before rebase:
main:    A──B──C
              \
feature:       D──E──F

After: git checkout feature && git rebase main
main:    A──B──C
                \
feature:         D'──E'──F'  ← replayed on top of C
```

**Decision Table:**

| Scenario | Use | Why |
|----------|-----|-----|
| Merging feature → main (via PR) | `merge` | Preserve branch history, no rewrite |
| Updating feature branch with main changes | `rebase` | Clean linear history on feature |
| Shared branch (develop, main) | `merge` | Never rewrite shared history |
| Your local feature branch | `rebase` | Clean history before PR |
| Audit trail required | `merge` | Merge commits show when branches joined |
| Open source PR contribution | `rebase` | Maintainers prefer linear history |

**Real commands:**
```bash
# Update feature branch with latest main (preferred for feature branches)
git checkout feature/user-auth
git fetch origin
git rebase origin/main
# Resolve any conflicts, then:
git rebase --continue

# Merge feature to main (after PR approval)
git checkout main
git merge --no-ff feature/user-auth   # --no-ff preserves merge commit
git push origin main
```
# Golden Rule: Never rebase commits that exist on origin/shared-branch

{{< /qa >}}


{{< qa num="4" q="Production has a critical bug. The fix is in a commit on your `develop` branch, but you need it on `main` NOW without merging all of develop. How do you do it?" level="advanced" >}}


**Answer:**

**Use `git cherry-pick`:**

```bash
# Step 1: Find the exact commit hash on develop
git log develop --oneline | grep "fix payment null"
# Output: a3f8d21 fix: payment null pointer exception in checkout

# Step 2: Cherry-pick to main
git checkout main
git pull origin main                          # Ensure you're up to date
git cherry-pick a3f8d21

# Step 3: Handle conflicts if any
git cherry-pick a3f8d21
# CONFLICT (content): Merge conflict in src/payment/checkout.java
git status                                    # See conflicted files
# ... resolve conflicts ...
git add src/payment/checkout.java
git cherry-pick --continue

# Step 4: Push and tag
git push origin main
git tag -a v2.1.1 -m "Hotfix: payment null pointer"
git push origin v2.1.1

# Cherry-pick a range of commits
git cherry-pick a3f8d21^..c7e9b45             # from a3f (inclusive) to c7e

# Cherry-pick without committing (stage only)
git cherry-pick --no-commit a3f8d21           # Useful to review before committing

# What actually happened:
# develop: A──B──[FIX]──D──E──F
# main:    A──X──Y──[FIX']      ← new commit, same change, different SHA
```
{{< /qa >}}



{{< qa num="5" q="You need to merge a long-running feature branch back to main after 3 months. The branch has 200+ commits and massive conflicts. What is your strategy?" level="advanced" >}}


**Answer:**

```bash
# Strategy: Incremental rebase + squash merge

# Step 1: Assess the damage
git log --oneline main..feature/big-refactor | wc -l   # Count commits: 247
git diff main...feature/big-refactor --stat             # See what changed

# Step 2: Update feature branch incrementally
git checkout feature/big-refactor
git fetch origin

# Rebase in chunks to catch conflicts early:
git rebase origin/main

# If too many conflicts, use merge instead:
git merge origin/main        # Creates merge commit, resolves conflicts once

# Step 3: Squash 200+ commits into logical units
git rebase -i $(git merge-base HEAD origin/main)
# In editor: squash/fixup WIP commits, keep logical ones:
# pick a1b2c3 feat: redesign auth module
# squash d4e5f6 WIP
# squash g7h8i9 fix typo
# pick j0k1l2 feat: add OAuth provider
# squash m3n4o5 fix OAuth tests

# Step 4: Final merge to main
git checkout main
git merge --no-ff feature/big-refactor -m "feat: big refactor — auth redesign + OAuth"
git push origin main

# Alternative: Squash merge (single commit)
git merge --squash feature/big-refactor
git commit -m "feat: 3-month auth refactor — redesign + OAuth + performance"
```
{{< /qa >}}


## 3. Undoing Changes & Recovery

{{< qa num="6" q="A developer ran `git push --force` on the `main` branch and overwrote 3 days of work. How do you recover it?" level="advanced" >}}


**Answer:**

```bash
# Step 1: Don't panic — git almost never permanently deletes data

# Step 2: Check reflog on the remote (if you have server access — GitHub/GitLab)
# GitHub: Settings → Branches → "Recover deleted branches" (within 30 days)
# GitLab: Admin Area → Repository → Recover lost commits

# Step 3: Check local reflogs of anyone who had the commits
# On Alice's machine (she had the latest code):
git reflog                    # Find the last good HEAD
# 9a8b7c6 HEAD@{0}: pull: Fast-forward
# f3e2d1c HEAD@{1}: commit: feat: add payment module   ← last good state
# ...

# Step 4: Create a recovery branch from the good state
git checkout -b recovery/force-push-recovery f3e2d1c
git push origin recovery/force-push-recovery

# Step 5: Restore main
git checkout main
git reset --hard recovery/force-push-recovery
git push --force-with-lease origin main    # Force push the recovery

# Step 6: Verify and clean up
git log --oneline -10
git branch -d recovery/force-push-recovery

# Prevention — set branch protection rules:
# GitHub: Settings → Branches → Add rule
# ✅ Require pull request reviews
# ✅ Restrict who can push to matching branches
# ✅ Require status checks
# ✅ Do not allow force pushes  ← THE KEY SETTING
# ✅ Do not allow deletions
```
{{< /qa >}}

{{< qa num="7" q="You accidentally committed sensitive data (passwords, API keys) and pushed to a public GitHub repo. What do you do?" level="advanced" >}}


**Answer:**

```bash
# ⚡ IMMEDIATE ACTIONS (first 5 minutes):

# 1. Revoke the exposed credentials FIRST (before anything else)
# AWS: IAM → Access Keys → Delete
# GitHub: Settings → Developer Settings → Personal Access Tokens → Revoke
# Stripe: Dashboard → Developers → API Keys → Roll

# 2. Make the repo private immediately (if possible)
# GitHub: Settings → Danger Zone → Change visibility → Private

# 3. Now clean the git history

# Option A: git filter-repo (recommended — faster than filter-branch)
pip install git-filter-repo

# Remove a specific file that contained secrets
git filter-repo --path config/secrets.env --invert-paths --force

# Remove specific string from all files in history
git filter-repo --replace-text <(echo 'AKIAIOSFODNN7EXAMPLE==>***REDACTED***') --force

# Option B: BFG Repo Cleaner (simpler for common cases)
java -jar bfg.jar --replace-text passwords.txt my-repo.git
# passwords.txt contains: PASSWORD=supersecret

# After cleaning — force push ALL branches and tags
git push origin --force --all
git push origin --force --tags

# 4. Notify GitHub to purge their caches
# Contact GitHub Support → request cache purge for your repo

# 5. Check if secrets were scraped (they likely were if exposed > 1 min)
# Search GitHub: https://github.com/search?q=AKIAIOSFODNN7EXAMPLE

# 6. Audit with Gitleaks to find any other secrets
gitleaks detect --source . --report-format json --report-path gitleaks-report.json

# Prevention:
# pre-commit hook with gitleaks or git-secrets
pre-commit install
# .pre-commit-config.yaml:
repos:
  - repo: https://github.com/zricethezav/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```
{{< /qa >}}


{{< qa num="8" q="Explain the difference between *git reset*, *git revert*, and *git restore* with real examples of when to use each." level="advanced" >}}

**Answer:**

```bash
# ─── GIT RESET ────────────────────────────────────────────────────────────────
# Moves HEAD (and optionally staging/working tree) to a previous commit
# ⚠️  REWRITES HISTORY — only use on LOCAL branches

# Scenario: You made 3 bad commits on your local feature branch
git log --oneline
# abc1234 bad commit 3
# def5678 bad commit 2
# ghi9012 bad commit 1
# jkl3456 last good commit

# --soft: Undo commits, keep changes STAGED
git reset --soft jkl3456
# → HEAD moved back, files still staged, ready to re-commit

# --mixed (default): Undo commits, keep changes in working tree (UNSTAGED)
git reset jkl3456
# → HEAD moved back, changes are unstaged (modified files)

# --hard: Undo commits AND discard all changes
git reset --hard jkl3456
# → HEAD moved back, working tree clean — DESTRUCTIVE!

# ─── GIT REVERT ───────────────────────────────────────────────────────────────
# Creates a NEW commit that undoes a previous commit
# ✅ SAFE for shared/public branches — does NOT rewrite history

# Scenario: Bad commit was pushed to main, teammates have pulled it
git log --oneline
# abc1234 feat: broken payment integration (this is the bad one)
# def5678 feat: add checkout page

git revert abc1234
# → Creates new commit: "Revert 'feat: broken payment integration'"
# → History: def5678 → abc1234 → [revert commit]

# Revert multiple commits
git revert HEAD~3..HEAD    # Revert last 3 commits

# ─── GIT RESTORE ──────────────────────────────────────────────────────────────
# Restores working tree files — does NOT touch commits or history

# Discard changes in working directory (unstaged)
git restore src/payment.java
# → File reverted to last committed state

# Unstage a file (opposite of git add)
git restore --staged src/payment.java

# Restore a file from a specific commit
git restore --source=HEAD~2 src/payment.java

# ─── DECISION CHART ───────────────────────────────────────────────────────────
# "I want to..."
# Unstage a file                     → git restore --staged <file>
# Discard working tree changes       → git restore <file>
# Undo local commits (not pushed)    → git reset --soft/mixed/hard
# Undo pushed commits (shared)       → git revert
# Go back and keep changes staged    → git reset --soft
```
{{< /qa >}}

## 4. Conflict Resolution

{{< qa num="9" q="Two developers edited the same file simultaneously. Walk through resolving a complex merge conflict professionally." level="intermediate" >}}


**Answer:**

```bash
# Scenario: Alice and Bob both edited src/auth/login.py

git merge feature/bob-auth
# CONFLICT (content): Merge conflict in src/auth/login.py
# Automatic merge failed; fix conflicts and then commit the result.

# Step 1: Understand what happened
git status                          # See all conflicted files
git log --merge --oneline           # See commits causing the conflict
git diff --merge                    # See all conflict diffs at once

# Step 2: Open conflict markers
cat src/auth/login.py
# <<<<<<< HEAD (Alice's version — current branch)
# def authenticate(user, password, mfa_token=None):
#     if mfa_token and not verify_mfa(user, mfa_token):
#         raise AuthError("MFA failed")
#     return db.verify_password(user, password)
# =======
# def authenticate(user, password):
#     audit_log(user, "login_attempt")
#     return db.verify_password(user, password)
# >>>>>>> feature/bob-auth (Bob's version — incoming)

# Step 3: Use a proper merge tool
git mergetool --tool=vimdiff     # Or: vscode, intellij, kdiff3

# Step 4: Manually merge the BEST of both:
# Combined solution — keep both MFA (Alice) AND audit logging (Bob)
def authenticate(user, password, mfa_token=None):
    audit_log(user, "login_attempt")             # Bob's addition
    if mfa_token and not verify_mfa(user, mfa_token):  # Alice's addition
        raise AuthError("MFA failed")
    return db.verify_password(user, password)

# Step 5: Mark resolved and commit
git add src/auth/login.py
git merge --continue
# Write a meaningful merge commit message:
# "Merge feature/bob-auth: combine MFA (Alice) + audit logging (Bob)"

# Step 6: Use rerere to remember this resolution
git config --global rerere.enabled true
# Next time the same conflict occurs, git resolves it automatically

# Step 7: Use diff3 style for better context
git config --global merge.conflictstyle diff3
# Shows BASE (common ancestor) + both versions
# <<<<<<< HEAD
# Alice's version
# ||||||| merged common ancestors
# ORIGINAL code
# =======
# Bob's version
# >>>>>>> feature/bob-auth
```
{{< /qa >}}


## 5. Git Internals & Architecture

{{< qa num="10" q="Explain what happens internally when you run `git commit`. Walk through the object model." level="intermediate" >}}


**Answer:**

```bash
# Git's 4 Object Types:
# blob    → file content
# tree    → directory listing (points to blobs and other trees)
# commit  → snapshot metadata (points to tree + parent commits)
# tag     → annotated pointer to a commit

# When you run: git commit -m "feat: add payment module"

# Step 1: Git hashes each staged file → blob objects
git cat-file -p HEAD:src/payment.java   # See blob content
# blob SHA: e69de29bb2d1d6434b8b29ae775ad8c2e48c5391

# Step 2: Git creates a tree object for each directory
git cat-file -p HEAD^{tree}
# 100644 blob e69de29b src/payment.java
# 100644 blob a3f8b7c2 src/checkout.java
# 040000 tree f1e2d3c4 tests/

# Step 3: Git creates the commit object
git cat-file -p HEAD
# tree    f8e7d6c5b4a3...    ← root tree
# parent  9a8b7c6d5e4f...    ← previous commit
# author  Alice <a@co.com> 1699000000 +0530
# committer Alice <a@co.com> 1699000000 +0530
#
# feat: add payment module

# Step 4: HEAD and branch ref updated
cat .git/HEAD                    # ref: refs/heads/main
cat .git/refs/heads/main         # a1b2c3d4e5... ← new commit SHA

# The full chain:
# HEAD → refs/heads/main → commit → tree → blobs

# Visualize object graph
git cat-file --batch-all-objects --batch-check | head -20

# Packfiles — git compresses objects
git gc                           # Packs loose objects into packfiles
ls .git/objects/pack/           # .pack + .idx files

# Content-addressable storage — SHA determines identity
echo "test content" | git hash-object --stdin
# 9daeafb9864cf43055ae93beb0afd6c7d144bfa4
```
{{< /qa >}}

## 6. Tagging & Releases


{{< qa num="11" q="How do you implement a professional versioning and tagging strategy with automated release notes?" level="advanced" >}}


**Answer:**

```bash
# Strategy: Conventional Commits + Semantic Versioning + Automated Changelog

# 1. Enforce Conventional Commits format
# feat:     → minor version bump (1.2.0 → 1.3.0)
# fix:      → patch version bump (1.2.0 → 1.2.1)
# feat!: or BREAKING CHANGE: → major bump (1.2.0 → 2.0.0)
# chore/docs/style → no version bump

# Commit examples:
git commit -m "feat(auth): add OAuth2 support with Google provider"
git commit -m "fix(payment): handle null card token in checkout"
git commit -m "feat!: redesign API response format — BREAKING CHANGE"

# 2. Automate with standard-version or semantic-release
npx semantic-release   # Reads commits, bumps version, generates CHANGELOG, pushes tag

# 3. Manual tagging (when needed)
git tag -a v2.3.0 -m "Release v2.3.0

Features:
- OAuth2 with Google (#123)
- Dark mode support (#145)

Bug Fixes:
- Payment null token crash (#167)
- Session timeout edge case (#171)"

git push origin v2.3.0

# 4. GitHub Actions — automated release pipeline
# .github/workflows/release.yml:
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0             # Need full history for changelog
      - uses: cycjimmy/semantic-release-action@v4
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

# 5. List and manage tags
git tag -l "v2.*"                    # List all v2.x tags
git show v2.3.0                      # Show tag details
git tag -d v2.3.0-bad                # Delete local tag
git push origin --delete v2.3.0-bad  # Delete remote tag

# Signed tags for security
git tag -s v2.3.0 -m "Signed release v2.3.0"   # GPG signed
git tag -v v2.3.0                               # Verify signature
```
{{< /qa >}}


## 7. Stash, Worktree & Advanced Commands

{{< qa num="12" q="You're in the middle of a feature when an urgent production bug comes in. How do you handle context switching cleanly?" level="intermediate" >}}


**Answer:**

```bash
# Option A: git stash (simple, same worktree)

# Save current work with a descriptive name
git stash push -m "WIP: OAuth integration — halfway through token refresh"
git stash push --include-untracked -m "WIP: OAuth — includes new config files"

# Switch to fix the bug
git checkout main
git pull origin main
git checkout -b hotfix/payment-crash
# ... fix the bug ...
git commit -m "fix: prevent null pointer in payment checkout"
git push origin hotfix/payment-crash
# Create PR → merge → done

# Return to your feature
git checkout feature/oauth-integration
git stash list
# stash@{0}: On feature/oauth: WIP: OAuth integration — halfway through token refresh
git stash pop                         # Restore + delete stash
# OR
git stash apply stash@{0}             # Restore but keep stash

# Option B: git worktree (better — separate directories, no stashing needed)

# Add a second worktree for the hotfix
git worktree add ../hotfix-workspace hotfix/payment-crash
cd ../hotfix-workspace
# Full working tree here — completely independent from main workspace
# Fix the bug...
git commit -m "fix: payment crash"
git push origin hotfix/payment-crash

# Return to your feature (original directory — untouched!)
cd ../main-project
# Continue exactly where you left off

# Clean up worktree when done
git worktree remove ../hotfix-workspace
git worktree list                     # Verify

# Option C: Quick commit (if work is commit-ready)
git add -A
git commit -m "WIP: OAuth token refresh — DO NOT MERGE"
git checkout -b hotfix/payment-crash main
# ... fix ...
git checkout feature/oauth
git reset HEAD~1                       # Undo the WIP commit, restore working state
```

{{< /qa >}}


{{< qa num="13" q="What is *git bisect* and when would a senior DevOps engineer use it? Show a real example." level="advanced" >}}


**Answer:**

```bash
# git bisect = binary search through commits to find which commit introduced a bug

# Scenario: App worked 2 weeks ago, broken today. 340 commits in between.
# Manual testing each = nightmare. git bisect = ~9 tests (log2(340) ≈ 8.4)

# Step 1: Start bisect
git bisect start

# Step 2: Mark known bad (current broken state)
git bisect bad HEAD

# Step 3: Mark known good (last known working state)
git bisect good v2.1.0          # OR use a commit hash

# Git checks out middle commit automatically:
# "Bisecting: 170 revisions left to test after this (roughly 8 steps)"

# Step 4: Test and mark each checkout
# (git checked out commit halfway between good and bad)
npm test                         # Run your test/check
git bisect good                  # This commit works → search upper half
# OR
git bisect bad                   # This commit is broken → search lower half

# Repeat until git says:
# "a3f8d21b is the first bad commit"
# commit a3f8d21b
# Author: Bob <bob@company.com>
# Date: Mon Oct 2 14:23:11 2023
# refactor: change payment provider initialization order

# Step 5: End bisect (restore HEAD)
git bisect reset

# Automated bisect with a test script
git bisect start
git bisect bad HEAD
git bisect good v2.1.0
git bisect run npm test          # Runs test automatically on each commit
# Git marks good/bad based on exit code (0 = good, non-zero = bad)

# Real use cases:
# - Performance regression (script that benchmarks and exits 1 if too slow)
# - Memory leak (script that checks memory usage)
# - API returning wrong data (script that curls endpoint and checks response)
# - Build failure (script that runs make/gradle/cargo)
```
{{< /qa >}}


## 8. Hooks & Automation


{{< qa num="14" q="Design a complete Git hooks strategy for enforcing code quality, preventing bad commits, and automating notifications." level="advanced" >}}


**Answer:**

```bash
# Hook Architecture:
# pre-commit     → lint, format, secret scanning, unit tests
# commit-msg     → enforce commit message format
# pre-push       → integration tests, branch protection
# post-merge     → install dependencies if lockfile changed
# post-checkout  → environment setup

# Tool: pre-commit framework (manages hooks across team)
pip install pre-commit

# .pre-commit-config.yaml (committed to repo):
repos:
  # Secret scanning
  - repo: https://github.com/zricethezav/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  # Python linting + formatting
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff
      - id: ruff-format

  # Conventional commits
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.0.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: [feat, fix, docs, style, refactor, test, chore, ci]

  # General checks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-merge-conflict
      - id: no-commit-to-branch
        args: ['--branch', 'main', '--branch', 'develop']

# Install hooks for all developers
pre-commit install                 # Installs pre-commit hook
pre-commit install --hook-type commit-msg   # Installs commit-msg hook

# Custom pre-push hook (.git/hooks/pre-push):
#!/bin/bash
echo "🧪 Running integration tests before push..."
npm run test:integration
if [ $? -ne 0 ]; then
  echo "❌ Integration tests failed. Push aborted."
  exit 1
fi
echo "✅ All tests passed. Pushing..."

# post-merge hook — auto-install dependencies
#!/bin/bash
CHANGED=$(git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD)
if echo "$CHANGED" | grep -q "package-lock.json"; then
  echo "📦 package-lock.json changed — running npm ci"
  npm ci
fi
if echo "$CHANGED" | grep -q "requirements.txt"; then
  echo "🐍 requirements.txt changed — running pip install"
  pip install -r requirements.txt --break-system-packages
fi

# Share hooks via the repo (not .git/hooks which isn't committed)
git config core.hooksPath .githooks/
# Put hook scripts in .githooks/ directory and commit them
```

{{< /qa >}}


## 9. Submodules & Subtrees

{{< qa num="15" q="When would you use git submodules vs git subtrees? Give a production scenario for each." level="advanced" >}}


**Answer:**

```bash
# ─── GIT SUBMODULES ───────────────────────────────────────────────────────────
# Best for: External dependencies you DON'T own / infrequently update
# Scenario: Your app depends on a company shared-config repo

# Add submodule
git submodule add https://github.com/company/shared-config.git config/shared
git commit -m "chore: add shared-config as submodule"

# Clone repo with submodules
git clone --recurse-submodules https://github.com/company/main-app.git
# OR for existing clone:
git submodule update --init --recursive

# Update submodule to latest
cd config/shared
git pull origin main
cd ../..
git add config/shared
git commit -m "chore: update shared-config to latest"

# Submodule Pros/Cons:
# ✅ Clear separation — submodule is its own repo with its own history
# ✅ Pinned to exact commit — reproducible builds
# ❌ Complex for contributors — easy to forget --recurse-submodules
# ❌ Merge conflicts in .gitmodules are confusing

# ─── GIT SUBTREES ─────────────────────────────────────────────────────────────
# Best for: Code you DO own and contribute back to / frequently merge
# Scenario: Merging a library repo INTO your monorepo, keeping history

# Add subtree (first time)
git subtree add --prefix=libs/payment \
  https://github.com/company/payment-lib.git main --squash

# Pull updates from the library
git subtree pull --prefix=libs/payment \
  https://github.com/company/payment-lib.git main --squash

# Push changes back to the library (contribute upstream)
git subtree push --prefix=libs/payment \
  https://github.com/company/payment-lib.git feature/new-provider

# Subtree Pros/Cons:
# ✅ No submodule complexity — just files in your repo
# ✅ Contributors don't need special commands
# ✅ Can contribute changes back upstream
# ❌ History is interleaved (can be messy with --squash to mitigate)
# ❌ Large repos get bloated if subtree is large

# Decision:
# External vendor lib, pinned version → Submodule
# Your own shared lib, need to contribute back → Subtree
# Very active shared code → Monorepo (skip both)
```
{{< /qa >}}


## 10. Large Repos & Performance

{{< qa num="16" q="Your Git repository has grown to 50GB and `git clone` takes 45 minutes. How do you fix this?" level="intermediate" >}}


**Answer:**

```bash
# Diagnose what's large
git count-objects -vH                     # Show object database stats
# size-pack: 48.3 GiB

# Find the largest objects
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectsize) %(rest)' \
  | awk '$1=="blob"' \
  | sort -k2 -rn \
  | head -20
# blob  2147483648  assets/videos/demo.mp4      ← 2GB video file!
# blob   536870912  data/training-dataset.zip    ← 512MB zip

# ─── SOLUTION 1: Git LFS (for new large files going forward) ───────────────
git lfs install
git lfs track "*.mp4" "*.zip" "*.psd" "*.tar.gz"
git add .gitattributes
git commit -m "chore: track large files with Git LFS"
# LFS stores pointer in git, actual file on LFS server

# ─── SOLUTION 2: Remove historical large files (BFG or filter-repo) ────────
# BFG — simpler
java -jar bfg.jar --strip-blobs-bigger-than 100M my-repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# filter-repo — more control
git filter-repo --path assets/videos --invert-paths

# ─── SOLUTION 3: Shallow clone for CI/CD ────────────────────────────────────
git clone --depth=1 https://github.com/company/big-repo.git
# Only clones latest commit — clone time: 45min → 30sec

# CI/CD pipeline (GitHub Actions):
- uses: actions/checkout@v4
  with:
    fetch-depth: 1                  # Shallow clone

# ─── SOLUTION 4: Sparse checkout (only clone what you need) ──────────────────
git clone --filter=blob:none --sparse https://github.com/company/monorepo.git
cd monorepo
git sparse-checkout set services/payment services/auth    # Only get these dirs
# Other directories are not downloaded

# ─── SOLUTION 5: Partial clone ────────────────────────────────────────────────
git clone --filter=blob:none https://github.com/company/big-repo.git
# Downloads tree/commits but not blobs until needed (lazy fetch)

# ─── SOLUTION 6: Maintenance for existing repos ───────────────────────────────
git maintenance start                  # Enables background maintenance
git gc --aggressive                    # Aggressive repacking
git repack -a -d --depth=250 --window=250
```
{{< /qa >}}


## 11. GitHub / GitLab Workflows

{{< qa num="17" q="A pull request has 47 commits with messages like *WIP*, *fix*, *fix2*, *please work*. How do you clean this up before merging to main?" level="advanced" >}}


**Answer:**

```bash
# Option A: Interactive Rebase (surgical control)
git checkout feature/messy-branch
git fetch origin

# Rebase against main to get clean base
git rebase origin/main

# Interactive rebase to squash/reword all commits
git rebase -i origin/main

# In editor — transform 47 commits:
# pick a1b2c3 feat: add OAuth login page          ← KEEP (rename if needed)
# squash b2c3d4 WIP                               ← squash into above
# squash c3d4e5 fix                               ← squash into above
# squash d4e5f6 fix2                              ← squash into above
# squash e5f6g7 please work                       ← squash into above
# pick f6g7h8 feat: add OAuth token refresh       ← KEEP
# squash g7h8i9 fix token expiry                  ← squash into above
# ...

# Write the final commit message when prompted:
# feat(auth): implement OAuth2 with Google
#
# - Add login page with Google OAuth button
# - Implement authorization code flow
# - Add token refresh logic with expiry handling
# - Write unit tests for all OAuth flows
#
# Closes #123

git push --force-with-lease origin feature/messy-branch
# Update the PR — now shows clean commits

# Option B: Squash Merge at PR time (GitHub/GitLab feature)
# GitHub: PR → "Squash and merge" button
# → Squashes all 47 commits into 1 automatically
# → You write the final commit message in the UI

# Option C: Reset + single commit
git reset origin/main                  # Unstage all commits (keep file changes)
git add -A
git commit -m "feat(auth): implement OAuth2 with Google

- Login page with Google OAuth button
- Authorization code flow
- Token refresh with expiry handling

Closes #123"
git push --force-with-lease origin feature/messy-branch

# Best practice:
# Set "Squash and merge" as the ONLY merge option in repo settings
# GitHub: Settings → General → Pull Requests → ✅ Allow squash merging only
```
{{< /qa >}}


## 12. Security & Secret Management

{{< qa num="18" q="How do you audit your entire git history for secrets across 200+ repos in your organization?" level="advanced" >}}


**Answer:**

```bash
# Tool: Gitleaks (fastest, most comprehensive)

# ─── SINGLE REPO SCAN ──────────────────────────────────────────────────────────
# Scan entire git history
gitleaks detect \
  --source . \
  --report-format sarif \
  --report-path gitleaks-report.sarif \
  --redact                           # Don't show actual secret values in report

# Scan only uncommitted changes
gitleaks protect --staged

# ─── ORGANIZATION-WIDE SCAN (200+ repos) ───────────────────────────────────────
# Using Gitleaks with GitHub API
gitleaks detect \
  --source https://github.com/myorg \
  --config gitleaks-org.toml \
  --report-format json \
  --report-path org-secrets-report.json

# Script to scan all repos:
#!/bin/bash
ORG="my-company"
TOKEN="$GITHUB_TOKEN"

# Get all repos
repos=$(gh repo list $ORG --limit 1000 --json name -q '.[].name')

for repo in $repos; do
  echo "Scanning $repo..."
  gh repo clone $ORG/$repo /tmp/scan/$repo -- --depth=1 2>/dev/null
  gitleaks detect \
    --source /tmp/scan/$repo \
    --report-format json \
    --report-path /tmp/reports/$repo.json \
    --no-git 2>/dev/null
  rm -rf /tmp/scan/$repo
done

# Aggregate reports
jq -s 'add' /tmp/reports/*.json > final-report.json

# ─── CI INTEGRATION (prevent future secrets) ────────────────────────────────────
# GitHub Actions:
- name: Gitleaks secret scan
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}

# ─── CONFIGURE CUSTOM RULES ────────────────────────────────────────────────────
# gitleaks-org.toml:
[[rules]]
  id = "company-internal-token"
  description = "Company internal API token"
  regex = '''CORP_[A-Z0-9]{32}'''
  tags = ["company", "token"]

[[allowlist]]
  description = "Allowlist known test credentials"
  regexes = ['''AKIAIOSFODNN7EXAMPLE''']   # AWS example key from docs

# ─── OTHER TOOLS ───────────────────────────────────────────────────────────────
# TruffleHog — entropy-based scanning
trufflehog git file://. --since-commit HEAD~100

# git-secrets (AWS)
git secrets --install
git secrets --register-aws
git secrets --scan-history

# GitHub Advanced Security — native, integrates with GitHub UI
# Enable: Settings → Security → Code scanning
```
{{< /qa >}}


## 13. Monorepo Strategies

{{< qa num="19" q="Your company is moving 15 microservices from separate repos into a monorepo. How do you do the migration while preserving all git history?" level="advanced" >}}


**Answer:**

```bash
# Goal: Merge 15 repos into 1 monorepo, preserving ALL commit history
# Each service gets its own subdirectory: /services/payment, /services/auth, etc.

# ─── METHOD: git subtree (preserves history) ────────────────────────────────────

# Step 1: Create the monorepo
git init company-monorepo
cd company-monorepo
git commit --allow-empty -m "chore: initialize monorepo"

# Step 2: For each service repo, rewrite history to add subdirectory prefix
# (so "main.py" becomes "services/payment/main.py" in the combined history)

for service in payment auth user notification inventory; do
  # Clone the service repo
  git clone https://github.com/company/$service-service.git /tmp/$service
  cd /tmp/$service

  # Rewrite history to prefix all paths with services/$service/
  git filter-repo --to-subdirectory-filter services/$service --force

  cd ~/company-monorepo

  # Add as remote and fetch
  git remote add $service /tmp/$service
  git fetch $service --no-tags

  # Merge into monorepo (allows unrelated histories)
  git merge --allow-unrelated-histories $service/main \
    -m "chore: import $service service history"

  # Clean up remote
  git remote remove $service
  echo "✅ Imported $service"
done

# Step 3: Push monorepo
git push origin main --force   # (Initial push)

# Step 4: Set up monorepo tooling
# Nx (Node.js ecosystem)
npx create-nx-workspace@latest company --preset=empty
# Turborepo
npx create-turbo@latest

# Step 5: CI optimization — only build affected services
# GitHub Actions with path filters:
on:
  push:
    paths:
      - 'services/payment/**'
jobs:
  payment-service:
    if: contains(github.event.commits.*.modified, 'services/payment')

# Nx affected commands:
npx nx affected:build          # Only build services affected by the changes
npx nx affected:test           # Only test affected services

# Verify history was preserved
git log --oneline services/payment/ | head -10
# Shows original commits from payment-service repo ✅
```
{{< /qa >}}


## 14. Troubleshooting Real Incidents

{{< qa num="20" q="During a deployment, git reports Your branch and *origin/main* have diverged. What happened and how do you fix it?" level="advanced" >}}


**Answer:**

```bash
# What happened:
# Your local main and remote main have DIFFERENT commits
# (Usually: someone force-pushed, or commits were amended on the remote)

git status
# On branch main
# Your branch and 'origin/main' have diverged,
# and have 3 and 5 commits each, respectively.

git log --oneline --left-right --graph main...origin/main
# < a1b2c3 (HEAD -> main) local commit you made
# < d4e5f6 another local commit
# < g7h8i9 yet another
# > j0k1l2 (origin/main) remote commit (different history)
# > m3n4o5 remote commit
# > ...

# ─── OPTION A: Remote is correct (someone rebased/force-pushed main) ─────────
# Safest: reset local to match remote
git fetch origin
git reset --hard origin/main
# ⚠️  Your 3 local commits are now orphaned (recoverable via reflog for 90 days)

# ─── OPTION B: Local is correct + remote has extra commits to keep ───────────
git pull --rebase origin main
# Replays your local commits ON TOP of the remote commits
# Result: clean linear history

# ─── OPTION C: Both have valid work to preserve ─────────────────────────────
git pull --no-rebase origin main   # Creates a merge commit
git push origin main

# ─── OPTION D: It's a CI pipeline — clean state needed ──────────────────────
git fetch origin main
git checkout -B main origin/main   # Force-reset local branch to remote
# This is safe in CI since there are no local commits to preserve

# Prevent this in CI:
- uses: actions/checkout@v4
  with:
    ref: main
    fetch-depth: 0
# actions/checkout always checks out clean state from remote
```
{{< /qa >}}


{{< qa num="21" q=" git pull is failing with SSL certificate errors in your CI pipeline. How do you debug and fix this properly?" level="advanced" >}}


**Answer:**

```bash
# Error symptoms:
# fatal: unable to access 'https://github.com/company/repo.git/'
# SSL certificate problem: unable to get local issuer certificate
# OR: SSL certificate problem: certificate has expired

# ─── DIAGNOSIS ──────────────────────────────────────────────────────────────
# Check what certificate is failing
curl -vI https://github.com 2>&1 | grep -E "SSL|certificate|issuer|expire"

# Check git SSL config
git config --list | grep ssl
# git config --global http.sslVerify false  ← NEVER DO THIS in production!

# ─── FIX 1: System CA bundle is outdated ────────────────────────────────────
# Ubuntu/Debian CI runner:
sudo apt-get update && sudo apt-get install -y ca-certificates
sudo update-ca-certificates

# Alpine (Docker):
apk add --no-cache ca-certificates && update-ca-certificates

# ─── FIX 2: Corporate proxy with custom CA cert (most common enterprise issue)
# Get the corporate CA cert from your security team → company-ca.pem

# Point git to the cert bundle
git config --global http.sslCAInfo /etc/ssl/certs/company-ca.pem

# OR add to system bundle
sudo cp company-ca.pem /usr/local/share/ca-certificates/company-ca.crt
sudo update-ca-certificates

# Environment variable approach (good for CI):
export GIT_SSL_CAINFO=/etc/ssl/certs/company-ca.pem
export CURL_CA_BUNDLE=/etc/ssl/certs/company-ca.pem
export NODE_EXTRA_CA_CERTS=/etc/ssl/certs/company-ca.pem   # For Node.js

# ─── FIX 3: Certificate expired on internal GitLab ──────────────────────────
# Renew the cert (contact your infra team)
# OR temporarily use SSH instead of HTTPS:
git remote set-url origin git@gitlab.company.com:group/repo.git

# ─── FIX 4: Self-signed cert on internal git server ─────────────────────────
# Get the self-signed cert
openssl s_client -connect gitlab.internal.com:443 </dev/null 2>/dev/null \
  | openssl x509 -outform PEM > /tmp/gitlab-cert.pem

git config --global http."https://gitlab.internal.com/".sslCAInfo /tmp/gitlab-cert.pem

# ─── NEVER DO THIS (insecure) ───────────────────────────────────────────────
# git config --global http.sslVerify false  ← Opens MITM attack vector

# Docker CI: add in Dockerfile
COPY company-gitlab-cert.pem /usr/local/share/ca-certificates/
RUN update-ca-certificates
```
{{< /qa >}}

{{< qa num="21" q="You run *git log* and notice commits are showing wrong author names/emails. How do you fix historical commits?" level="advanced" >}}


**Answer:**

```bash
# Check the damage
git log --format="%h %ae %an" | sort -k2 | uniq -c -f1 | sort -rn
# 47 bob@personal.com Bob Smith    ← personal email used by mistake
# 156 bob@company.com Bob Smith    ← correct

# Fix author for all commits with wrong email
git filter-repo \
  --email-callback 'return email.replace(b"bob@personal.com", b"bob@company.com")' \
  --name-callback 'return name.replace(b"Bobby", b"Bob Smith")' \
  --force

# Verify
git log --format="%ae" | sort | uniq -c

# Alternative: amend just the last commit
git commit --amend --author="Bob Smith <bob@company.com>" --no-edit

# Fix author for last N commits
git rebase -i HEAD~10
# Change 'pick' to 'edit' for commits you want to change
# For each:
git commit --amend --author="Bob Smith <bob@company.com>" --no-edit
git rebase --continue

# Prevent this: set correct identity per repo or globally
git config --global user.name "Bob Smith"
git config --global user.email "bob@company.com"

# Per-repo override (for open source contributions):
git config user.email "bob@personal.com"

# Enforce correct email via hook:
# .git/hooks/pre-commit:
EXPECTED_DOMAIN="company.com"
CURRENT_EMAIL=$(git config user.email)
if [[ "$CURRENT_EMAIL" != *"@${EXPECTED_DOMAIN}" ]]; then
  echo "❌ Wrong email: $CURRENT_EMAIL"
  echo "Set: git config user.email you@${EXPECTED_DOMAIN}"
  exit 1
fi
```
{{< /qa >}}


## 💡 Git Command Quick-Reference for Senior Engineers

```bash
# ─── DAILY WORKFLOW ──────────────────────────────────────────────────────────
git status -s                              # short status
git log --oneline --graph --all            # visual branch history
git diff --staged                          # review staged changes before commit
git commit --amend --no-edit               # add to last commit without new message
git push --force-with-lease               # force push SAFELY (fails if remote changed)

# ─── BRANCH MANAGEMENT ───────────────────────────────────────────────────────
git branch -vv                             # list branches with tracking info
git branch --merged main | grep -v main   # find branches merged into main (safe to delete)
git branch --no-merged main               # branches NOT yet merged
git remote prune origin                   # remove stale remote-tracking branches
git fetch --prune                         # fetch + prune in one step

# ─── HISTORY INSPECTION ──────────────────────────────────────────────────────
git log --author="Alice" --since="2 weeks" --oneline
git log -S "payment" --oneline            # find commits that added/removed "payment"
git log -G "def authenticate" --oneline   # find commits where regex matched diff
git blame -L 10,20 src/auth.py            # who wrote lines 10-20 of auth.py
git log --follow -p src/auth.py           # full history of a file, following renames

# ─── RECOVERY ────────────────────────────────────────────────────────────────
git reflog                                 # every HEAD movement in last 90 days
git checkout HEAD~3 -- src/payment.java    # restore a file from 3 commits ago
git cherry-pick sha1^..sha2               # cherry-pick a range of commits
git stash list && git stash show -p       # list and inspect stashes

# ─── REPO MAINTENANCE ────────────────────────────────────────────────────────
git gc --aggressive --prune=now           # compress objects, free space
git fsck --full                           # check repo integrity
git remote -v                             # list all remotes
git shortlog -sn --all                    # contributor stats

# ─── ALIASES (put in ~/.gitconfig) ───────────────────────────────────────────
[alias]
  lg = log --oneline --graph --all --decorate
  st = status -s
  last = log -1 HEAD --stat
  undo = reset HEAD~1 --mixed
  unstage = restore --staged
  aliases = config --get-regexp alias
  whoops = commit --amend --no-edit
  pf = push --force-with-lease
```

---

## 🔧 `.gitconfig` for Senior DevOps Engineers

```ini
[user]
    name = Your Name
    email = you@company.com
    signingkey = YOUR_GPG_KEY_ID

[core]
    editor = vim
    autocrlf = input             # LF on commit (Unix-style)
    whitespace = trailing-space,space-before-tab
    pager = delta                # Use delta for beautiful diffs

[commit]
    gpgsign = true               # Sign all commits with GPG

[merge]
    conflictstyle = diff3        # Show base version in conflicts
    tool = vimdiff

[pull]
    rebase = true                # git pull always rebases

[push]
    default = current            # push to same-name remote branch
    autoSetupRemote = true       # auto --set-upstream on first push

[rebase]
    autoStash = true             # stash before rebase, pop after
    autoSquash = true            # auto-apply fixup! commits

[rerere]
    enabled = true               # remember conflict resolutions (huge time saver!)

[fetch]
    prune = true                 # auto-remove stale remote branches
    pruneTags = true

[log]
    date = relative              # "2 hours ago" instead of timestamp

[diff]
    colorMoved = default
    algorithm = histogram        # better diff algorithm

[interactive]
    diffFilter = delta --color-only

[delta]
    navigate = true
    light = false
    side-by-side = true
    line-numbers = true

[alias]
    lg = log --oneline --graph --all --decorate
    st = status -s
    pf = push --force-with-lease
    undo = reset HEAD~1 --mixed
    whoops = commit --amend --no-edit
    gone = "!git fetch -p && git branch -vv | grep 'origin/.*: gone]' | awk '{print $1}' | xargs git branch -D"
```

---

## 📚 Resources

| Resource | Purpose |
|----------|---------|
| [Pro Git Book (free)](https://git-scm.com/book/en/v2) | Definitive Git reference |
| [Conventional Commits](https://www.conventionalcommits.org/) | Commit message standard |
| [git-filter-repo](https://github.com/newren/git-filter-repo) | History rewriting tool |
| [git-lfs](https://git-lfs.com/) | Large file storage |
| [pre-commit](https://pre-commit.com/) | Hook management framework |
| [Gitleaks](https://gitleaks.io/) | Secret scanning |
| [delta](https://dandavison.github.io/delta/) | Beautiful git diffs |
| [lazygit](https://github.com/jesseduffield/lazygit) | TUI for git |
| [gitui](https://github.com/extrawurst/gitui) | Fast TUI in Rust |

---

## 🏆 Key Principles Senior Interviewers Test For

1. **History is sacred** — Know when to rebase vs merge, never `--force` on shared branches.
2. **Security mindset** — Pre-commit hooks, signed commits, secret scanning, no credentials in code.
3. **Recovery skills** — `reflog`, `bisect`, `cherry-pick` — can you recover from disasters?
4. **Team scalability** — Branching strategy appropriate for team size and release cadence.
5. **Automation** — Hooks, CI/CD triggers, automated changelog from commit messages.
6. **Monorepo vs polyrepo** — Know trade-offs and migration strategies.
7. **Performance** — Shallow clone, sparse checkout, LFS for large repos.
8. **Internals** — Object model (blob, tree, commit, tag), packfiles, reflog mechanics.

---

*Last updated: 2026 | Real scenarios sourced from production incidents at high-growth startups and enterprise engineering teams.*

> ⭐ Complete your Senior DevOps preparation with the AWS, Python, Docker, and Jenkins README files in this series!

</div>
