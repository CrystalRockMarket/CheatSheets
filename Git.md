# Git

## Table of Contents
1. [Introduction](#introduction)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Commands](#basic-commands)
4. [Branching and Merging](#branching-and-merging)
5. [Remote Operations](#remote-operations)
6. [Stashing and Cleaning](#stashing-and-cleaning)
7. [Undo and Reset](#undo-and-reset)
8. [History and Inspection](#history-and-inspection)
9. [Tagging and Releases](#tagging-and-releases)
10. [Submodules](#submodules)
11. [Configuration](#configuration)
12. [Advanced Topics](#advanced-topics)
13. [Quick Reference](#quick-reference)

---

## Introduction

Git is a distributed version control system for tracking changes in source code during software development.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Git Architecture                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Working Directory                     │   │
│   │                   (your project files)                   │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                      Staging Area (Index)                │   │
│   │                 (about to be committed)                  │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Local Repository                      │   │
│   │              (.git directory - full history)             │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌───────────────────┐ │   │
│   │  │   Objects   │  │   Refs      │  │    HEAD           │ │   │
│   │  │  (blobs,    │  │  (branches, │  │  (current ref)    │ │   │
│   │  │  trees,     │  │   tags)     │  │                   │ │   │
│   │  │  commits)   │  │             │  │                   │ │   │
│   │  └─────────────┘  └─────────────┘  └───────────────────┘ │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│              ┌─────────────┴─────────────┐                       │
│              ▼                           ▼                       │
│   ┌────────────────────────┐  ┌────────────────────────┐        │
│   │    Remote Repository   │  │    Your Fork/Clone     │        │
│   │   (GitHub, GitLab,     │  │   (full copy locally)  │        │
│   │    Bitbucket, etc.)    │  │                        │        │
│   └────────────────────────┘  └────────────────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Git Object Model

```
┌─────────────────────────────────────────────────────────────┐
│                      Commit Graph                            │
│                                                              │
│   A ── B ── C ── D ── E (main)                              │
│              │                                               │
│              └── F ── G (feature)                            │
│                                                              │
│   A, B, C, D, E, F, G = Commits (SHA-1 hashes)              │
│   main, feature = Branch references (pointers)              │
│   HEAD = Current branch/commit                              │
│                                                              │
│   Each commit contains:                                      │
│   - Tree (snapshot of files)                                │
│   - Parent commit(s)                                        │
│   - Author and committer                                    │
│   - Timestamp                                               │
│   - Commit message                                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Installation and Setup

### Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install git

# RHEL/CentOS/Fedora
sudo yum install git
sudo dnf install git

# Arch Linux
sudo pacman -S git

# macOS
brew install git

# Windows
# Download from https://git-scm.com/download/win
choco install git

# From source
git clone https://github.com/git/git.git
cd git
make prefix=/usr all
sudo make prefix=/usr install
```

### Initial Setup

```bash
# Set username
git config --global user.name "Your Name"

# Set email
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor vim
git config --global core.editor "code --wait"

# Set default branch name
git config --global init.defaultBranch main

# Enable color output
git config --global color.ui auto

# Enable credential helper
git config --global credential.helper cache
git config --global credential.helper store
git config --global credential.helper osxkeychain  # macOS

# List all settings
git config --list --show-origin

# Get specific setting
git config user.name
git config --get remote.origin.url
```

### Setup SSH Keys for GitHub

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub

# Add to GitHub:
# Settings → SSH and GPG keys → New SSH key

# Test connection
ssh -T
```

---

 git@github.com## Basic Commands

### Initialize Repository

```bash
# Initialize new repository
git init

# Initialize in specific directory
git init /path/to/project

# Initialize with custom branch name
git init -b main

# Clone existing repository
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git

# Clone with specific directory name
git clone https://github.com/user/repo.git myproject

# Clone specific branch
git clone -b branch-name https://github.com/user/repo.git

# Clone shallow (limited history)
git clone --depth 1 https://github.com/user/repo.git

# Clone with all branches
git clone --mirror https://github.com/user/repo.git
```

### Basic Workflow

```bash
# Check status
git status

# Stage changes
git add filename
git add .                    # Stage all changes
git add -A                   # Stage all (new, modified, deleted)
git add -u                   # Stage modified and deleted only
git add -p                   # Interactive staging

# Commit changes
git commit -m "Commit message"
git commit -am "Message"     # Stage + commit (tracked files only)

# Commit with detailed message
git commit
# Opens editor for multi-line message

# Amend last commit
git commit --amend
git commit --amend --no-edit
git commit --amend -m "New message"

# Amend with author
git commit --amend --author="Name <email@example.com>"

# View staged/unstaged changes
git diff                    # Unstaged changes
git diff --cached           # Staged changes
git diff HEAD               # All changes vs HEAD

# View changes in specific file
git diff filename
git diff --stat filename

# Show staged and unstaged in one command
git diff HEAD --stat
```

### File Operations

```bash
# Remove file from git and disk
git rm filename

# Remove from git only (keep on disk)
git rm --cached filename

# Remove directory
git rm -r directory/

# Rename file
git mv oldname newname

# Restore file to last commit
git checkout -- filename
git restore --source=HEAD filename

# Restore to specific version
git restore --source=HEAD~2 filename

# Restore deleted file
git restore --source=HEAD filename

# View file at specific commit
git show HEAD:filename
git show commit-hash:filename
```

---

## Branching and Merging

### Branch Management

```bash
# List branches
git branch

# List all branches (including remote)
git branch -a

# List remote branches only
git branch -r

# List branches with last commit
git branch -v
git branch -vv

# Create new branch
git branch branch-name

# Create and switch to new branch
git checkout -b branch-name
git switch -c branch-name

# Switch branch
git checkout branch-name
git switch branch-name

# Switch to previous branch
git checkout -
git switch -

# Delete branch
git branch -d branch-name      # Safe delete
git branch -D branch-name      # Force delete

# Delete remote branch
git push origin --delete branch-name
git push origin :branch-name

# Rename branch
git branch -m old-name new-name
git branch -M old-name new-name   # Force rename

# Track remote branch
git branch --set-upstream-to=origin/branch-name
git branch -u origin/branch-name

# List tracking branches
git branch -vv
```

### Merging

```bash
# Merge branch into current branch
git merge branch-name

# Merge with no fast-forward (creates merge commit)
git merge --no-ff branch-name

# Merge with fast-forward only
git merge --ff-only branch-name

# Merge with custom message
git merge -m "Merge branch 'feature'"

# Merge specific commit
git cherry-pick commit-hash

# Merge without committing
git merge --no-commit branch-name

# Abort merge
git merge --abort

# Merge with squash
git merge --squash branch-name
git commit -m "Merge feature branch"

# Find common ancestor
git merge-base main feature
```

### Rebasing

```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase
git rebase -i HEAD~n
git rebase -i --root

# Rebase with specific commit
git rebase --onto new-base current-base

# Continue rebase after fixing
git rebase --continue

# Skip rebase patch
git rebase --skip

# Abort rebase
git rebase --abort

# Rebase with autosquash
git rebase -i --autosquash

# Rebase preserve merges
git rebase -p branch-name

# Pull with rebase instead of merge
git pull --rebase
git config --global pull.rebase true
```

### Merge Conflicts

```bash
# During merge, conflicts occur
git merge branch-name
# CONFLICT (content): Merge conflict in filename

# Check conflict status
git status

# View conflicts
git diff
git diff --name-only --diff-filter=U

# Resolve conflict manually
# Edit the file, fix the conflict markers

# Use merge tool
git mergetool
git mergetool --tool=vimdiff

# Accept all changes from one side
git checkout --ours filename       # Current branch
git checkout --theirs filename     # Merged branch

# Mark resolved
git add filename

# Continue merge
git commit

# Abort merge
git merge --abort
```

---

## Remote Operations

### Remote Management

```bash
# List remotes
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git

# Add remote with SSH
git remote add origin git@github.com:user/repo.git

# Rename remote
git remote rename origin upstream

# Change remote URL
git remote set-url origin https://github.com/user/repo.git
git remote set-url origin git@github.com:user/repo.git

# Remove remote
git remote remove origin

# Show remote details
git remote show origin

# Prune remote-tracking branches
git remote prune origin

# Set default remote for current branch
git branch --set-upstream-to=origin/branch-name
```

### Fetching and Pulling

```bash
# Fetch from remote
git fetch
git fetch origin
git fetch --all

# Fetch specific branch
git fetch origin branch-name

# Fetch and prune
git fetch --prune

# Fetch all remotes
git fetch --multiple origin upstream

# Pull (fetch + merge)
git pull
git pull origin main
git pull --rebase

# Pull with specific strategy
git pull -X theirs     # Prefer their changes
git pull -X ours       # Prefer our changes

# Pull without automatic merge
git pull --ff-only
```

### Pushing

```bash
# Push to remote
git push
git push origin main
git push origin branch-name

# Push to specific remote
git push upstream branch-name

# Push with tracking
git push -u origin branch-name

# Push tags
git push --tags
git push origin v1.0.0

# Push single tag
git push origin tag-name
git push origin --delete tag-name

# Force push
git push --force
git push -f origin branch-name

# Force with lease (safer)
git push --force-with-lease
git push --force-with-lease origin branch-name

# Delete remote branch
git push origin --delete branch-name

# Push to multiple remotes
git remote set-url --add origin https://github/user/repo.git
git remote set-url --add --push origin https://github/user/repo.git

# Push to bare repository
git push /path/to/bare/repo.git main
```

---

## Stashing and Cleaning

### Stashing

```bash
# Stash uncommitted changes
git stash

# Stash with message
git stash save "WIP: feature in progress"

# Stash untracked files
git stash -u
git stash --include-untracked

# Stash all including ignored
git stash -a

# List stashes
git stash list

# Show stash contents
git stash show
git stash show -p

# Apply last stash
git stash apply
git stash apply stash@{0}

# Apply and drop stash
git stash pop
git stash pop stash@{0}

# Drop stash
git stash drop
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-branch-name
```

### Cleaning

```bash
# Show what would be deleted
git clean -n
git clean --dry-run

# Remove untracked files
git clean -f

# Remove untracked directories
git clean -fd

# Remove ignored files
git clean -fX

# Remove everything (untracked + ignored)
git clean -f

# Interactive clean
git clean -i
```

---

## Undo and Reset

### Reset Types

```bash
# Soft reset (keep changes staged)
git reset --soft HEAD~1
git reset --soft commit-hash

# Mixed reset (default - unstage changes)
git reset HEAD~1
git reset --mixed HEAD~1
git reset commit-hash

# Hard reset (discard all changes)
git reset --hard HEAD~1
git reset --hard commit-hash

# Reset specific file
git reset HEAD filename
git reset -- filename

# Reset to remote state
git reset --hard origin/main

# Reset with preserve uncommitted
git reset --keep HEAD~1
```

### Reverting

```bash
# Create new commit that undoes changes
git revert commit-hash

# Revert without auto-commit
git revert --no-commit commit-hash
git commit

# Revert merge commit
git revert -m 1 commit-hash     # Revert to parent 1 (main)
git revert -m 2 commit-hash     # Revert to parent 2 (feature)

# Revert range of commits
git revert HEAD~3..HEAD

# Revert with edit
git revert -e commit-hash
```

### Checkout vs Reset vs Revert

| Command | Safe? | Changes History | Use Case |
|---------|-------|-----------------|----------|
| `git checkout` | Yes | No | Switch branches, files |
| `git reset` | No | Yes | Undo commits locally |
| `git revert` | Yes | No | Undo commits publicly |
| `git restore` | Yes | No | Restore files (Git 2.23+) |

### Restore (Git 2.23+)

```bash
# Restore file to last commit
git restore filename

# Restore to specific commit
git restore --source=HEAD~2 filename

# Restore and stage
git restore --staged filename

# Restore from specific branch
git restore --source=branch-name filename
```

---

## History and Inspection

### Viewing History

```bash
# View commit log
git log

# View with patches
git log -p

# View limited commits
git log -n 5
git log --oneline

# View with graph
git log --graph --oneline

# View with branches
git log --all --graph --oneline

# View author stats
git log --author="Name"

# View by date
git log --since="2024-01-01"
git log --until="2024-12-31"

# View file history
git log --follow filename

# View specific changes
git log -p -- filename

# View blame
git blame filename
git blame -L 10,20 filename

# Show specific commit
git show
git show commit-hash
git show HEAD~3:filename

# Show short log
git shortlog
git shortlog -sn
```

### Searching

```bash
# Search in commit messages
git log --grep="fix"
git log --grep="fix" --oneline

# Search in changes
git log -S "search-term"
git log -S "search-term" --oneline

# Search with regex
git log --grep="^feat"
git log --grep="^fix" --all-match

# Search in specific files
git log -- "*.js"
git log -- "*.py" --since="2024-01-01"

# Grep in working tree
git grep "search-term"
git grep -n "search-term"

# Grep in specific commit
git grep "search-term" commit-hash
```

### Diff Commands

```bash
# View changes
git diff
git diff --staged
git diff HEAD

# Diff between branches
git diff main...feature
git diff main feature

# Diff specific file
git diff filename
git diff --cached filename

# Show diff statistics
git diff --stat
git diff --stat main feature

# Diff with word changes
git diff --word-diff

# Show diff for specific commit
git show commit-hash --stat
git show commit-hash -p
```

### Log Formats

```bash
# Pretty formats
git log --pretty=format:"%h - %s"
git log --pretty=full
git log --pretty=fuller
git log --oneline
git log --graph --all

# Custom format
git log --pretty=format:"%h|%an|%ad|%s" --date=short

# Placeholders:
# %h - short hash
# %H - full hash
# %an - author name
# %ae - author email
# %ad - author date
# %ar - author relative date
# %s - subject
# %b - body
# %d - refs
```

---

## Tagging and Releases

### Tag Management

```bash
# List tags
git tag
git tag -l "v1.*"

# Create lightweight tag
git tag v1.0.0

# Create annotated tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Create tag with GPG
git tag -s v1.0.0 -m "Signed release"
git tag -v v1.0.0               # Verify tag

# Tag specific commit
git tag v1.0.0 commit-hash

# Tag with checksum
git tag -a v1.0.0 -u "Key ID" -m "Message"

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0

# Push tag
git push origin v1.0.0
git push origin --tags

# List tags with details
git tag -n
git tag -l -n "v1.*"
```

### Tag Comparison

```bash
# Show tag differences
git diff v1.0.0 v1.1.0

# Show commits since tag
git log v1.0.0..HEAD

# Show files in tag
git ls-tree v1.0.0

# Show tag info
git show v1.0.0
```

### Release Workflow

```bash
# Create release branch
git checkout -b release/v1.0.0 main

# Update version
echo "1.0.0" > VERSION
git add VERSION
git commit -m "Bump version to 1.0.0"

# Create release tag
git tag -a v1.0.0 -m "Release 1.0.0"

# Push to remote
git push origin main
git push origin release/v1.0.0
git push origin v1.0.0

# Merge back to main
git checkout main
git merge release/v1.0.0 --no-ff
git push origin main

# Delete release branch
git branch -d release/v1.0.0
git push origin --delete release/v1.0.0
```

---

## Submodules

### Adding Submodules

```bash
# Add submodule
git submodule add https://github.com/user/lib.git

# Add with specific branch
git submodule add -b main https://github.com/user/lib.git

# Add with custom directory name
git submodule add https://github.com/user/lib.git lib-name

# Initialize after clone
git submodule update --init --recursive
```

### Managing Submodules

```bash
# Clone repository with submodules
git clone --recursive https://github.com/user/repo.git

# Initialize existing submodules
git submodule update --init

# Update submodules
git submodule update --remote

# Update specific submodule
git submodule update --remote lib-name

# Update and merge
git submodule update --remote --merge

# Update with rebase
git submodule update --remote --rebase

# List submodules
git submodule status

# Set submodule URL
git config submodule.lib.url https://github.com/user/newlib.git

# Set submodule branch
git config -f .gitmodules submodule.lib.branch develop
git submodule set-branch -b develop lib
```

### Working with Submodules

```bash
# Enter submodule
cd lib-name
git checkout develop
git pull

# Commit in submodule
cd lib-name
git add .
git commit -m "Update lib"
git push

# Return to main repo
cd ..
git add lib-name
git commit -m "Update lib to latest"

# Status with submodules
git submodule status

# Diff submodule
git diff lib-name

# Deinit submodule
git submodule deinit lib-name
git rm lib-name
```

---

## Configuration

### Git Config Levels

```bash
# Local (repository) - .git/config
git config --local user.name "Name"

# Global (user) - ~/.gitconfig
git config --global user.name "Name"

# System (all users) - /etc/gitconfig
git config --system user.name "Name"

# List all config
git config --list --show-origin
```

### Useful Configurations

```bash
# Identity
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# Editor
git config --global core.editor vim
git config --global core.editor "code --wait"

# Diff tool
git config --global merge.tool vimdiff
git config --global diff.tool vimdiff

# Credential helper
git config --global credential.helper cache
git config --global credential.helper store
git config --global credential.helper osxkeychain

# Aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.last 'log -1 HEAD'
git config --global alias.unstage 'reset HEAD --'
git config --global alias.visual '!gitk'

# Pretty logs
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# Branch defaults
git config --global init.defaultBranch main
git config --global push.default current
git config --global pull.rebase false

# Fetch and prune
git config --global fetch.prune true

# Rebase settings
git config --global rebase.autostash true
git config --global rebase.autosquash true

# Useful settings
git config --global core.autocrlf input
git config --global core.sshCommand "ssh -i ~/.ssh/id_ed25519"
```

### Config File Locations

```bash
# System config
/etc/gitconfig

# Global config
~/.gitconfig

# Local config
.git/config

# Gitignore (per repo)
.gitignore

# Gitattributes (per repo)
.gitattributes
```

---

## Advanced Topics

### Interactive Rebase

```bash
# Start interactive rebase
git rebase -i HEAD~5

# Commands in editor:
# pick - use commit
# reword - use commit, edit message
# edit - stop to amend
# squash - combine with previous
# fixup - combine, discard message
# drop - remove commit
# exec - run command
```

### Bisect

```bash
# Start bisect
git bisect start

# Mark known good commit
git bisect good commit-hash

# Mark known bad commit
git bisect bad commit-hash

# Git checks out middle commit
# Test, then:
git bisect good  # or bad

# Continue until found
# End bisect
git bisect reset
```

### Worktrees

```bash
# Create worktree
git worktree add ../feature-branch main

# List worktrees
git worktree list

# Remove worktree
git worktree remove ../feature-branch

# Prune stale worktrees
git worktree prune
```

### Refspec

```bash
# Fetch all refs
git config --add remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'

# Fetch specific branch
git config --add remote.origin.fetch '+refs/pull/*:refs/pull/*'

# Push refspec
git push origin main:refs/heads/main
```

### Bundle

```bash
# Create bundle
git bundle create repo.bundle main

# Verify bundle
git bundle verify repo.bundle

# Clone from bundle
git clone repo.bundle repo

# List bundles
git bundle list-heads repo.bundle
```

### Git Large File Storage (LFS)

```bash
# Install
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.zip"

# .gitattributes
*.psd filter=lfs diff=lfs merge=lfs -text

# Normal workflow
git add .
git commit
git push

# Check tracked files
git lfs ls-files
```

### Garbage Collection

```bash
# Clean up loose objects
git gc

# Aggressive cleanup
git gc --aggressive

# Prune unreachable objects
git prune

# Verify object connectivity
git fsck

# Show repository size
du -sh .git
git count-objects -v
```

---

## Quick Reference

### Common Commands

| Command | Description |
|---------|-------------|
| `git init` | Initialize repo |
| `git clone` | Clone repo |
| `git add` | Stage changes |
| `git commit` | Commit |
| `git push` | Push to remote |
| `git pull` | Pull from remote |
| `git fetch` | Fetch changes |
| `git checkout` | Switch branch |
| `git branch` | List/create branches |
| `git merge` | Merge branches |
| `git status` | Show status |
| `git diff` | Show changes |
| `git log` | Show history |

### File States

| State | Description |
|-------|-------------|
| Modified | Changed in working dir |
| Staged | Added to index |
| Committed | In HEAD |

### Reset Scope

| Command | Working Dir | Index | HEAD |
|---------|-------------|-------|------|
| `--soft` | Kept | Kept | Changed |
| `--mixed` | Kept | Changed | Changed |
| `--hard` | Changed | Changed | Changed |

### Merge Strategies

| Strategy | Description |
|----------|-------------|
| `recursive` | Default for 3-way merge |
| `resolve` | Two-way merge |
| `ours` | Keep ours |
| `theirs` | Keep theirs |
| `octopus` | Multiple branches |

### Useful Aliases

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.last 'log -1 HEAD'
git config --global alias.unstage 'reset HEAD --'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

---

## See Also

- [Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book)
- [GitHub Guides](https://guides.github.com)
- [GitLab Documentation](https://docs.gitlab.com)
- [Oh My Git](https://ohmygit.org)
