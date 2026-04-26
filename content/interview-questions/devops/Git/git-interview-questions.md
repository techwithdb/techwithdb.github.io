---
title: "Git Interview Questions and Answers (2026) Part 01"
description: "30 real-world Git scenario-based interview questions and answers covering branching strategies, merge/rebase, undoing changes, conflict resolution, Git internals, hooks, submodules, security, monorepos, and troubleshooting — Senior DevOps Engineer Edition."
date: 2026-04-16T14:26:30+05:30
author: "DB"
tags: ["Git", "DevOps", "CI/CD", "Interview", "Version Control", "GitHub", "GitLab"]
tool: "git"
level: "All Levels"
question_count: 30
draft: false
---

<div class="qa-list">

## Basic Interview Questions

{{< qa num="1" q="What is `origin` in Git?" level="basic" >}}

In Git, `origin` refers to the default name given to the remote repository from which the local repository was cloned. It is used as a reference to control fetches, pulls, and pushes.

{{< /qa >}}

{{< qa num="2" q="What is the purpose of the `.gitignore` file?" level="basic" >}}

The `.gitignore` file tells Git which files and folders to ignore when tracking changes. It prevents unnecessary files — such as logs, temporary files, or compiled artifacts — from being committed to the repository, keeping it clean and focused on important source files only.

{{< /qa >}}

{{< qa num="3" q="What is a Version Control System (VCS)?" level="basic" >}}

A Version Control System (VCS) records the work of developers collaborating on projects. It maintains a history of code changes, allowing developers to add features, fix bugs, and run tests safely. If required, they can restore a previous working version, ensuring project integrity.

{{< /qa >}}

{{< qa num="4" q="What are the advantages of using Git?" level="basic" >}}

Using Git provides several advantages:

1. Supports teamwork by allowing multiple developers to collaborate on the same project simultaneously.
2. Each developer holds a full local copy of the repository, improving performance and enabling offline work.
3. Free, open-source, and widely supported across platforms.
4. Suitable for projects of all sizes and types.
5. Each repository contains a single `.git` directory managing the entire history.

{{< /qa >}}

{{< qa num="5" q="What is a `git conflict`?" level="basic" >}}

Git usually handles merges automatically, but conflicts arise when:

- Two branches edit the **same line** of a file differently.
- One branch **deletes a file** that another branch has modified.

In such cases, Git cannot decide which change to keep and requires manual resolution.

{{< /qa >}}

{{< qa num="6" q="What is the meaning of `Index` in Git?" level="basic" >}}

In Git, the **Index** (also called the **Staging Area**) is a temporary holding space for changes before they are committed to the repository. It allows you to selectively prepare specific modifications from your working directory before permanently saving them as part of the project history.

{{< /qa >}}

{{< qa num="7" q="How do you change the last commit in Git?" level="basic" >}}

Use `git commit --amend` to modify the most recent commit:

1. Make your changes to the files.
2. Stage them with `git add`.
3. Run the amend command:

```bash
git commit --amend
```

Git will open your default editor to update the commit message. Save and close to apply.

> **Note:** Avoid amending commits that have already been pushed to a shared remote, as it rewrites history.

{{< /qa >}}

{{< qa num="8" q="How do you rename a branch in Git?" level="basic" >}}

- **Rename the current branch:**

```bash
git branch -m <new_branch_name>
```

- **Rename a different branch:**

```bash
git branch -m <old_branch_name> <new_branch_name>
```

- **Push the renamed branch to remote and update tracking:**

```bash
git push origin -u <new_branch_name>
git push origin --delete <old_branch_name>
```

{{< /qa >}}

---

## Intermediate Interview Questions

{{< qa num="9" q="What is the difference between `git fetch` and `git pull`?" level="intermediate" >}}

| Feature | `git fetch` | `git pull` |
|---|---|---|
| Purpose | Downloads changes from remote but doesn't apply them | Downloads **and merges** changes into your current branch |
| Working Directory | No changes — safe to inspect before merging | Changes are applied immediately |
| Use Case | When you want to review changes before merging | When you want to update your branch quickly |
| Workflow | Often followed by `git merge` or `git rebase` | Combines `fetch` + `merge` in one step |

**Using `git fetch` with manual merge:**

```bash
git fetch origin
git log origin/main       # inspect incoming changes
git merge origin/main     # apply after review
```

**Using `git pull` directly:**

```bash
git pull origin main
```

{{< /qa >}}

{{< qa num="10" q="What are the benefits of using a Pull Request in a project?" level="intermediate" >}}

Pull Requests (PRs) are a core part of collaborative development:

- **Code Review:** Team members can review code before it is merged, ensuring higher quality and catching bugs early.
- **Collaboration:** Developers can discuss, comment, and suggest changes within the PR itself.
- **Version Control:** Tracks proposed changes clearly and keeps the main branch stable.
- **Accountability:** Maintains a clear record of who approved and merged each change.

{{< /qa >}}

{{< qa num="11" q="What is `git stash`? How is it used?" level="intermediate" >}}

`git stash` temporarily saves uncommitted changes so you can switch context — such as moving to another branch — without losing your in-progress work. Stashed changes can be reapplied later.

| Command | Purpose |
|---|---|
| `git stash` | Save current uncommitted changes |
| `git stash list` | View all saved stashes |
| `git stash pop` | Reapply the most recent stash and remove it from stash list |
| `git stash apply` | Reapply a stash without removing it from history |
| `git stash drop stash@{n}` | Delete a specific stash entry |

{{< /qa >}}

{{< qa num="12" q="How do you revert a commit that has already been pushed and made public?" level="intermediate" >}}

Use `git revert` — it creates a **new commit** that undoes the changes, preserving history:

1. **Switch to the target branch:**

```bash
git checkout <branch_name>
```

2. **Find the commit hash:**

```bash
git log
```

3. **Revert the commit:**

```bash
git revert <commit-hash>
```

4. Git opens an editor to confirm the revert message. Save and close.

5. **Push the revert commit:**

```bash
git push origin <branch_name>
```

> **Why `git revert` over `git reset`?** Revert is safe for shared repositories because it does not rewrite history.

{{< /qa >}}

{{< qa num="13" q="Explain the difference between `git revert` and `git reset`?" level="intermediate" >}}

| Parameter | `git reset` | `git revert` |
|---|---|---|
| History | Moves HEAD to a previous commit, **removing** commits from history | Creates a **new commit** that undoes changes, keeping history intact |
| Safety | Risky on shared/public branches | Safe to use on shared repositories |
| Use Case | Undoing local, unpushed commits | Undoing commits that are already pushed |

{{< /qa >}}

{{< qa num="14" q="What is the difference between `git reflog` and `git log`?" level="intermediate" >}}

| `git reflog` | `git log` |
|---|---|
| Tracks all movements of HEAD and branch pointer changes | Displays the commit history of the current branch |
| Shows references even if they are no longer part of any branch | Only shows commits reachable from the current branch |
| Can help **recover lost commits** or deleted branches | Does not track changes outside the commit chain |
| Useful for disaster recovery and debugging | Used for reviewing past changes and audit trails |

{{< /qa >}}

{{< qa num="15" q="What is `HEAD` in Git?" level="intermediate" >}}

`HEAD` is a special pointer in Git that always refers to the **current position** in the repository:

- Points to the **latest commit** on the currently checked-out branch.
- When you switch branches, HEAD automatically updates to the tip of the new branch.
- A **detached HEAD** state occurs when HEAD points directly to a specific commit instead of a branch — common when checking out a tag or old commit.

{{< /qa >}}

{{< qa num="16" q="What is the purpose of `git tag -a`?" level="intermediate" >}}

`git tag -a` creates an **annotated tag**, which stores additional metadata alongside the tag reference:

- Tagger's name and email
- Date and time of tagging
- A custom descriptive message

**Create an annotated tag:**

```bash
git tag -a v1.0 -m "Version 1.0 Release"
```

**Push the tag to remote:**

```bash
git push origin v1.0
```

Annotated tags are preferred over lightweight tags for marking official releases because of their stored metadata.

{{< /qa >}}

{{< qa num="17" q="What is the difference between `HEAD`, working tree, and index in Git?" level="intermediate" >}}

| Concept | HEAD | Working Tree | Index (Staging Area) |
|---|---|---|---|
| Definition | Reference to the current commit or branch | Directory containing your actual project files | Buffer between working tree and repository |
| Contains | Pointer stored in `.git/HEAD` | Unstaged edits and new files | Changes staged with `git add`, ready to commit |
| Location | `.git/HEAD` | Your local project folder | `.git/index` |

{{< /qa >}}

{{< qa num="18" q="How do you resolve a `git conflict`?" level="intermediate" >}}

1. **Identify conflicting files:**

```bash
git status
```

2. **Open the conflicted file.** Git marks conflicts like this:

```
<<<<<<< HEAD
Your changes
=======
Incoming changes
>>>>>>> feature-branch
```

3. **Manually edit the file** to keep your changes, the incoming changes, or a combination of both. Remove all conflict markers.

4. **Stage the resolved file:**

```bash
git add <file>
```

5. **Commit the merge:**

```bash
git commit
```

6. **If rebasing**, continue with:

```bash
git rebase --continue
```

{{< /qa >}}

{{< qa num="19" q="Explain the difference between `git merge` and `git rebase`, and when you would use each?" level="intermediate" >}}

| # | `git merge` | `git rebase` |
|---|---|---|
| How it works | Combines two branches and creates a **merge commit** | Reapplies commits from one branch **on top** of another |
| History | Preserves both branches' full history | Rewrites commit history to produce a **linear** timeline |
| Merge commit | Yes — a merge commit is created | No — history is rewritten without merge commits |
| Best for | Collaborative branches where history matters | Personal branches or cleanup before opening a PR |

**Rule of thumb:** Use `merge` to preserve context in team workflows; use `rebase` to maintain a clean, readable history on feature branches.

{{< /qa >}}

{{< qa num="20" q="How do you delete a remote Git branch?" level="intermediate" >}}

```bash
git push origin --delete <branch_name>
```

To also remove the local tracking reference:

```bash
git fetch --prune
```

{{< /qa >}}

{{< qa num="21" q="What is the purpose of `git cherry-pick`?" level="intermediate" >}}

`git cherry-pick` applies changes from a **specific commit** to the current branch without merging the entire source branch.

Key use cases:

- **Selective commit application:** Pick only the commits you need.
- **Bug fixes:** Backport a specific fix to another branch (e.g., a release branch).
- **No full branch merge required:** Incorporate isolated changes cleanly.
- **New commit:** The picked changes are applied as a **new commit** on the current branch.

```bash
git cherry-pick <commit-hash>
```

{{< /qa >}}

---

## Advanced Interview Questions

{{< qa num="22" q="What is a Git hook and how might you use one?" level="advanced" >}}

A **Git hook** is a script that runs automatically at specific points in the Git workflow. They live inside `.git/hooks/` and can be written in any scripting language.

| Hook | Trigger | Common Use |
|---|---|---|
| `pre-commit` | Before a commit is created | Run linters or unit tests |
| `commit-msg` | After commit message is written | Enforce commit message format |
| `post-commit` | After a commit is completed | Trigger notifications or logging |
| `pre-push` | Before pushing to remote | Run full test suite |

**Example — `pre-commit` hook to run ESLint:**

```bash
#!/bin/sh
npx eslint . || exit 1
```

Make hooks executable:

```bash
chmod +x .git/hooks/pre-commit
```

For team-wide hooks, consider tools like **Husky** to version-control them alongside your project.

{{< /qa >}}

{{< qa num="23" q="How can you undo a commit that hasn't been pushed to the remote repository?" level="advanced" >}}

Use `git reset` with the appropriate flag depending on what you want to keep:

- **Keep changes staged (soft reset):**

```bash
git reset --soft HEAD~1
```

- **Keep changes in working directory but unstaged (mixed reset — default):**

```bash
git reset --mixed HEAD~1
```

- **Discard all changes completely (hard reset):**

```bash
git reset --hard HEAD~1
```

> Use `--hard` with caution — changes are permanently lost unless recoverable via `git reflog`.

{{< /qa >}}

{{< qa num="24" q="How do you squash multiple commits into one using interactive rebase?" level="advanced" >}}

1. **Start an interactive rebase** for the last N commits:

```bash
git rebase -i HEAD~<number-of-commits>
```

2. In the editor, leave the first commit as `pick` and change subsequent ones to `squash` (or `s`):

```
pick   a1b2c3d  First commit message
squash e4f5g6h  Second commit message
squash i7j8k9l  Third commit message
```

3. Save and close. Git opens another editor to combine the commit messages — edit as needed and save.

4. Force-push if the branch was already pushed:

```bash
git push origin <branch_name> --force-with-lease
```

{{< /qa >}}

{{< qa num="25" q="How do you reset to a previous commit without losing changes in the working directory?" level="advanced" >}}

Use a **soft reset** to move HEAD back while leaving your files and staged changes intact:

```bash
git reset --soft <commit-hash>
```

Your commits are undone, but all the changes remain staged — ready to be re-committed or reorganized.

{{< /qa >}}

{{< qa num="26" q="What is the difference between `git reset --hard` and `git clean -fd`?" level="advanced" >}}

| Command | What it resets | Affects tracked files | Affects untracked files |
|---|---|---|---|
| `git reset --hard` | HEAD, index, and working directory to a specific commit | ✅ Yes — discards modifications | ❌ No — leaves untracked files untouched |
| `git clean -fd` | Only the working directory | ❌ No — ignores tracked files | ✅ Yes — removes untracked files and directories |

**Combined usage** — to fully clean a repository:

```bash
git reset --hard HEAD
git clean -fd
```

{{< /qa >}}

{{< qa num="27" q="Explain the Git workflow and the lifecycle of a file?" level="advanced" >}}

A file in Git moves through four primary states:

| State | Description |
|---|---|
| **Untracked** | Newly created file not yet known to Git |
| **Modified** | File has been edited since the last commit |
| **Staged** | File added via `git add`, queued for the next commit |
| **Committed** | File permanently saved to repository history via `git commit` |

**Full lifecycle example:**

```bash
# Create a new file
touch feature.js

# Stage it
git add feature.js

# Commit it
git commit -m "Add feature.js"

# Modify it
vim feature.js

# Stage and commit the update
git add feature.js
git commit -m "Update feature.js"
```

Each `git commit` creates an immutable snapshot of all staged files at that point in time.

{{< /qa >}}

{{< qa num="28" q="Explain the three types of `git reset`?" level="advanced" >}}

Git provides three reset modes, each with a different scope of impact:

| Type | Command | Behaviour |
|---|---|---|
| **Soft** | `git reset --soft <commit>` | Moves HEAD only — index and working directory remain unchanged. Changes stay staged. |
| **Mixed** | `git reset --mixed <commit>` | Moves HEAD and clears the index — changes remain in the working directory as unstaged. |
| **Hard** | `git reset --hard <commit>` | Moves HEAD, clears the index, **and** discards working directory changes completely. |

**Decision guide:**
- Use `--soft` to re-commit with a different message or grouping.
- Use `--mixed` to unstage changes and re-evaluate what to commit.
- Use `--hard` only when you are certain you want to throw away all changes.

{{< /qa >}}

{{< qa num="29" q="What are Git Tags and why are they important?" level="advanced" >}}

Git **tags** are named pointers to specific commits — most commonly used to mark **release points** (e.g., `v1.0`, `v2.1`). Unlike branches, tags do not move; they permanently reference an exact commit.

**Types of tags:**

| Type | Description |
|---|---|
| **Lightweight** | A simple pointer to a commit, no extra metadata |
| **Annotated** | Stores tagger name, email, date, and a message — recommended for releases |

**Common tag commands:**

| Command | Purpose |
|---|---|
| `git tag v1.0` | Create a lightweight tag |
| `git tag -a v2.0 -m "Release 2.0"` | Create an annotated tag |
| `git tag` | List all tags |
| `git show v2.0` | View tag details |
| `git push origin v2.0` | Push a specific tag to remote |
| `git push origin --tags` | Push all local tags to remote |

{{< /qa >}}

{{< qa num="30" q="How do you use `git bisect` to find a bug-introducing commit?" level="advanced" >}}

`git bisect` uses **binary search** through commit history to efficiently pinpoint the commit that introduced a bug — without manually checking each one.

1. **Start bisecting:**

```bash
git bisect start
```

2. **Mark the current (bad) commit:**

```bash
git bisect bad
```

3. **Mark a known good commit:**

```bash
git bisect good <commit-hash>
```

4. Git checks out a commit halfway between. **Test your code**, then tell Git the result:

```bash
git bisect good   # if the bug is not present
git bisect bad    # if the bug is present
```

5. Repeat until Git identifies the exact commit that introduced the bug.

6. **End the session:**

```bash
git bisect reset
```

> For large repositories, `git bisect` can find the culprit commit in `O(log n)` steps — dramatically faster than manual search.

{{< /qa >}}

</div>
