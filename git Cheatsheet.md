# Git Cheatsheet

## Daily operations

```bash
# get something
git clone git@github.com:<USER>/<REPO>.git [<target-dir>]

# Check what's going on
git status

# Stage a file
git add filename

# Stage everything
git add .

# Commit with message
git commit -m "description of change"

# Pull latest from remote
git pull

# Push to remote
git push

## Setup & Config

```bash
# Set your identity (once per machine)
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# Check current config
git config --list

# Set default branch name to "main" for new repos
git config --global init.defaultBranch main

# Set default editor (e.g. vi, nano, code)
git config --global core.editor vi
```

## Getting Started

```bash
# Clone a repo
git clone git@github.com:USER/REPO.git

# Clone into a specific folder
git clone git@github.com:USER/REPO.git my-folder

# Initialize a new repo in current directory
git init
```

## Daily Workflow

```bash
# Check what's going on
git status

# See what's changed (unstaged)
git diff

# See what's staged (ready to commit)
git diff --staged

# Stage a file
git add filename

# Stage everything
git add .

# Commit with message
git commit -m "description of change"

# Stage + commit all tracked files in one go
git commit -am "description of change"

# Pull latest from remote
git pull

# Push to remote
git push
```

## Viewing History

```bash
# Show commit log
git log

# Compact one-line log
git log --oneline

# Show log with a graph (useful with branches)
git log --oneline --graph --all

# Show what changed in last commit
git show

# Show what changed in a specific commit
git show abc1234

# Who changed what in a file (blame)
git blame filename
```

## Undoing Things

```bash
# Unstage a file (keep changes in working dir)
git restore --staged filename

# Discard changes in working dir (CAREFUL - gone forever)
git restore filename

# Amend the last commit message (only if not pushed yet!)
git commit --amend -m "new message"

# Amend the last commit with additional file changes
git add forgotten-file
git commit --amend --no-edit

# Undo last commit but keep changes staged
git reset --soft HEAD~1

# Undo last commit and unstage changes (keep files)
git reset HEAD~1

# Undo last commit and discard everything (DANGEROUS)
git reset --hard HEAD~1

# Revert a commit (creates a new commit that undoes it - safe for pushed stuff)
git revert abc1234
```

**Rule of thumb:** Use `revert` for anything already pushed. Use `reset` only for local, unpushed commits.

## Branches

```bash
# List branches (* = current)
git branch

# List all branches including remote
git branch -a

# Create a new branch
git branch feature-name

# Switch to a branch
git checkout feature-name
# or (newer syntax):
git switch feature-name

# Create + switch in one go
git checkout -b feature-name
# or:
git switch -c feature-name

# Delete a branch (only if merged)
git branch -d feature-name

# Force delete a branch (even if not merged - CAREFUL)
git branch -D feature-name

# Push a new branch to remote
git push -u origin feature-name
```

## Merging

```bash
# Merge a branch into current branch
git merge feature-name

# Abort a merge if there are conflicts you don't want to deal with right now
git merge --abort
```

**When you get merge conflicts:** Git marks the conflicting sections in the file with `<<<<<<<`, `=======`, and `>>>>>>>`. Edit the file to keep what you want, remove the markers, then `git add` and `git commit`.

## Rebase (The Basics)

Rebase replays your commits on top of another branch. It makes history cleaner (linear) but **rewrites commits**, so: **never rebase stuff that's already pushed/shared**.

```bash
# Rebase current branch onto main
git rebase main

# If conflicts arise during rebase:
git rebase --continue    # after fixing conflicts
git rebase --abort       # nope, forget it

# Interactive rebase: squash/reorder/edit last N commits
git rebase -i HEAD~3
```

**When to use:** Before merging a feature branch, to get a clean linear history. When in doubt, just use `merge` — it's safer.

## Stash (Temporary Shelf)

```bash
# Stash current changes (working dir goes back to clean)
git stash

# Stash with a description
git stash push -m "work in progress on feature X"

# List stashes
git stash list

# Apply most recent stash (keeps it in stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply a specific stash
git stash apply stash@{2}

# Drop a specific stash
git stash drop stash@{0}

# Drop all stashes
git stash clear
```

## Remotes

```bash
# Show remotes
git remote -v

# Change remote URL (e.g. for deploy key aliases)
git remote set-url origin git@my-alias:USER/REPO.git

# Add a second remote
git remote add upstream git@github.com:OTHER_USER/REPO.git

# Fetch from a remote without merging
git fetch origin
```

## Deploy Keys & SSH Config

Each deploy key is unique to one repo. To use multiple repos with deploy keys on the same machine:

1. Generate a key per repo:
```bash
ssh-keygen -t ed25519 -C "deploy-REPONAME" -f ~/.ssh/deploy_REPONAME
```

2. Add host aliases to `~/.ssh/config`:
```
Host github-REPONAME
    HostName github.com
    User git
    IdentityFile ~/.ssh/deploy_REPONAME
    IdentitiesOnly yes
```

3. Set the remote to use the alias:
```bash
git remote set-url origin git@github-REPONAME:USER/REPO.git
```

SSH sees the alias, looks up the real hostname and the correct key from the config, and connects transparently.

Don't forget `chmod 600` on the private keys.

## Tags

```bash
# List tags
git tag

# Create a lightweight tag
git tag v1.0

# Create an annotated tag (recommended)
git tag -a v1.0 -m "Version 1.0 release"

# Push a tag to remote
git push origin v1.0

# Push all tags
git push --tags

# Delete a local tag
git tag -d v1.0

# Delete a remote tag
git push origin --delete v1.0
```

## .gitignore

Create a `.gitignore` file in the repo root:

```
# Example .gitignore
node_modules/
*.pyc
__pycache__/
.env
.DS_Store
*.log
```

```bash
# If you already tracked a file and want to stop:
git rm --cached filename
```

## Useful Aliases

Add to `~/.gitconfig` or via `git config --global`:

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all"
```

Then use `git st`, `git co`, `git lg`, etc.

## Oh No, I Messed Up

| Situation | Fix |
|---|---|
| Committed to wrong branch | `git reset HEAD~1`, switch branch, commit there |
| Pushed something I shouldn't have | `git revert abc1234` and push |
| Need to find a lost commit | `git reflog` — shows everything, even "deleted" stuff |
| Everything is on fire | `git reflog` is your friend. Commits are rarely truly gone. |

## Quick Reference: vi Find & Replace

Since you'll be editing commit messages and config files:

| Command | What it does |
|---|---|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` / `N` | Next / previous match |
| `:s/old/new/` | Replace first on current line |
| `:s/old/new/g` | Replace all on current line |
| `:%s/old/new/g` | Replace all in file |
| `:%s/old/new/gc` | Replace all, ask each time |
