# git-quick-reference

A short git command reference, grouped by task, with a pre-push secret check for Linux, macOS, and Windows
PowerShell. Placeholders in angle brackets stand for your own values.

- [docs/SECRET-CHECKLIST.md](docs/SECRET-CHECKLIST.md): what to check before every push, and why.
- Ready-to-run preflight scripts and the A/B/C/D deploy runbook format are in
  [cv-security-best-practice](https://github.com/cv-ai-sec/cv-security-best-practice/tree/main/quick-reference).

## Getting and creating projects

| Command | What it does |
|---|---|
| `git init -b main` | Start a new repo in the current folder, with `main` as the first branch |
| `git clone <url>` | Copy an existing remote repo locally |
| `git remote add origin <url>` | Point a local repo at a remote |
| `git remote -v` | List the configured remotes and their URLs |

## Basic snapshotting

| Command | What it does |
|---|---|
| `git status` | Show staged, unstaged, and untracked changes. Run it before every `add`. |
| `git add <file> [<file> ...]` | Stage specific files. Name them, and avoid `git add -A` and `git add .`, so you know exactly what is staged. |
| `git diff` | Show unstaged changes |
| `git diff --cached` | Show staged changes not yet committed |
| `git commit -m "<message>"` | Record staged changes as a commit |
| `git restore <file>` | Discard unstaged changes to a file |
| `git restore --staged <file>` | Unstage a file without discarding its changes |

## Branching and merging

| Command | What it does |
|---|---|
| `git branch` | List local branches |
| `git branch <name>` | Create a branch |
| `git switch -c <name>` | Create and switch to a new branch |
| `git switch <name>` | Switch to an existing branch |
| `git merge <branch>` | Merge another branch into the current one |
| `git branch -M main` | Rename the current branch to `main` |

## Sharing and updating

| Command | What it does |
|---|---|
| `git fetch origin` | Download remote changes without merging them |
| `git pull` | Fetch, then merge the remote branch into the current one |
| `git push` | Upload local commits to the remote |
| `git push -u origin main` | Push, and set `origin/main` as the upstream (first push only) |
| `git rebase origin/main` | Replay local commits on top of the remote's latest, instead of adding a merge commit |

## Inspection and comparison

| Command | What it does |
|---|---|
| `git log --oneline` | Compact commit list |
| `git log --oneline origin/main..HEAD` | Commits that exist locally but not on the remote: what a push would send |
| `git diff origin/main..HEAD` | The full diff a push would send. Read it, not only the commit messages. |
| `git diff origin/main..HEAD --stat` | The same, as a file list with line counts |
| `git show <commit>` | The diff of one commit |
| `git ls-files` | Every file git tracks. Use it to confirm a `.env` was never added. |
| `git check-ignore -v <path>` | Show which ignore rule matches a path |

## Pre-push secret check

Run it before every push, not only the first. The reasoning for each step is in
[docs/SECRET-CHECKLIST.md](docs/SECRET-CHECKLIST.md).

### Linux and macOS (bash)

```bash
# 1. Re-check what is staged
git status
git diff --cached --stat

# 2. Search the staged diff for secret-shaped strings
git diff --cached | grep -iE "(api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY)"
# Any hit: stop and inspect that line. Placeholders and variable names are fine; real values are not.

# 3. Confirm no real .env is tracked (should print nothing, or only .env.example)
git ls-files | grep -E "(^|/)\.env"

# 4. Confirm the ignore file is respected (should print a rule)
git check-ignore -v .env

# 5. Final scan of everything about to be pushed
git diff origin/main..HEAD | grep -iE "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"
git log --oneline origin/main..HEAD
```

### Windows PowerShell

```powershell
# 1. Re-check what is staged
git status
git diff --cached --stat

# 2. Search the staged diff for secret-shaped strings
git diff --cached | Select-String -Pattern "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"

# 3. Confirm no real .env is tracked (should print nothing, or only .env.example)
git ls-files | Select-String "(^|/)\.env"

# 4. Final scan of everything about to be pushed
git diff origin/main..HEAD | Select-String -Pattern "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"
git log --oneline origin/main..HEAD
```
