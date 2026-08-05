# Git Cheat Sheet

> A comprehensive reference covering Git setup, everyday workflows, branching, remotes, history, undo operations, and advanced techniques.

---

# Setup

## Concepts

**Repository (Repo)**
A repository is a folder tracked by Git. It stores your files, history, branches, and metadata.

```text
Project Folder
│
├── Files
├── Folders
└── .git/  <-- Git database and history
```

**Working Directory**
The files you actively edit on your machine.

**Staging Area (Index)**
A preparation area where changes are collected before creating a commit.

```text
Working Directory
       |
       v
 Staging Area
       |
       v
     Commit
```

**Commit**
A snapshot of your project at a specific point in time.

## Initial Configuration

```bash
git config --global user.name "Your Name"      # Set author name
git config --global user.email "you@email.com" # Set author email
```

```bash
git config --global init.defaultBranch main     # Set default branch name
git config --global core.editor "code --wait"  # Set default editor
```

```bash
git config --list                               # Display active configuration
```

## Initialize or Clone

```bash
git init                                        # Create a new Git repository
```

```bash
git clone https://github.com/user/repo.git      # Clone a remote repository
```

---

# Basics

## Concepts

**Tracked File**
A file Git already knows about and monitors.

**Untracked File**
A file that exists locally but is not yet tracked by Git.

**Stage**
The act of selecting changes that will be included in the next commit.

## Daily Workflow

```text
Edit Files
    |
    v
git add
    |
    v
git commit
    |
    v
git push
```

## Status and Inspection

```bash
git status                                      # Show file status and branch info
```

```bash
git diff                                        # Compare working directory vs staged version
```

```bash
git diff --staged                               # Compare staged changes vs last commit
```

## Staging Changes

```bash
git add file.txt                                # Stage one file
git add .                                       # Stage all changes in current directory
```

## Committing Changes

```bash
git commit -m "Add login feature"              # Create commit with message
```

```bash
git commit -am "Update existing files"         # Stage tracked files and commit
```

## Ignoring Files

```bash
# .gitignore
*.log                                           # Ignore log files
node_modules/                                   # Ignore dependency directory
.env                                             # Ignore environment files
```

---

# Branching

## Concepts

**Branch**
An independent line of development.

```text
main
 |
 +----- feature-login
 |
 +----- bugfix-header
```

**Merge**
Combines changes from one branch into another.

**Merge Conflict**
Occurs when Git cannot automatically decide which changes to keep.

## Branch Operations

```bash
git branch                                      # List local branches
```

```bash
git branch feature-login                        # Create new branch
```

```bash
git switch feature-login                        # Switch branch
```

```bash
git switch -c feature-login                     # Create and switch
```

```bash
git branch -d feature-login                     # Delete merged branch
```

> **Warning:** Deleting a branch removes the branch reference. Ensure its commits are merged or otherwise recoverable.

## Merging

```bash
git switch main                                 # Move to target branch
git merge feature-login                         # Merge source branch
```

## Rebase

**Rebase**
Moves commits onto a new base, creating a cleaner linear history.

```text
Before:
A---B main
 \
  C---D feature

After Rebase:
A---B main
     \
      C'--D' feature
```

```bash
git switch feature-login                        # Go to feature branch
git rebase main                                 # Reapply commits onto main
```

> **Warning:** Avoid rebasing commits already shared with other developers unless the team agrees.

## Merge vs Rebase

| Operation | Purpose | History Style |
|-----------|---------|--------------|
| Merge | Combine branches | Preserves branch structure |
| Rebase | Replay commits on new base | Linear history |

---

# Remote

## Concepts

**Remote Repository**
A repository hosted elsewhere such as GitHub, GitLab, or Azure DevOps.

**Origin**
The default remote name created during cloning.

```text
Local Repo <-----> Remote Repo
      push/pull
```

## Remote Management

```bash
git remote -v                                   # Show configured remotes
```

```bash
git remote add origin https://repo-url.git      # Add remote repository
```

```bash
git remote remove origin                        # Remove remote definition
```

## Push and Pull

```bash
git push origin main                            # Upload commits
```

```bash
git push -u origin main                         # Set upstream and push
```

```bash
git pull origin main                            # Fetch and merge changes
```

```bash
git fetch origin                                # Download changes only
```

## Fetch vs Pull

| Command | Downloads Changes | Merges Automatically |
|----------|-------------------|---------------------|
| Fetch | Yes | No |
| Pull | Yes | Yes |

---

# History & Undo

## Concepts

**HEAD**
Pointer to the current commit.

```text
HEAD
 |
 v
Commit C
 |
Commit B
 |
Commit A
```

**Reset**
Moves branch pointers and optionally changes files.

**Revert**
Creates a new commit that undoes a previous commit.

## Viewing History

```bash
git log                                         # Full commit history
```

```bash
git log --oneline --graph --decorate            # Compact visual history
```

## Restore Files

```bash
git restore file.txt                            # Restore working copy
```

> **Warning:** Uncommitted changes may be lost.

## Undo Staging

```bash
git restore --staged file.txt                   # Remove file from staging area
```

## Revert Commit

```bash
git revert COMMIT_HASH                          # Create reverse commit
```

## Reset Options

```bash
git reset --soft HEAD~1                         # Undo commit, keep staging
```

```bash
git reset --mixed HEAD~1                        # Undo commit, keep files
```

```bash
git reset --hard HEAD~1                         # Remove commit and file changes
```

> **Warning:** git reset --hard permanently discards local changes that are not otherwise recoverable.

## Reset Comparison

| Reset Type | Commit Removed | Changes Kept | Staging Preserved |
|-----------|---------------|--------------|-------------------|
| soft | Yes | Yes | Yes |
| mixed | Yes | Yes | No |
| hard | Yes | No | No |

---

# Advanced

## Concepts

**Cherry-Pick**
Copies specific commits from one branch to another.

**Stash**
Temporarily stores unfinished work.

**Tag**
Named marker typically used for releases.

## Stashing

```bash
git stash                                       # Store current changes
```

```bash
git stash list                                  # View stashes
```

```bash
git stash pop                                   # Restore latest stash
```

## Cherry Pick

```bash
git cherry-pick COMMIT_HASH                     # Apply selected commit
```

## Tags

```bash
git tag v1.0.0                                  # Create lightweight tag
```

```bash
git tag -a v1.0.0 -m "Release 1.0"             # Create annotated tag
```

```bash
git push origin --tags                          # Push all tags
```

## Interactive Rebase

```bash
git rebase -i HEAD~5                            # Edit last 5 commits
```

> **Warning:** Rewriting commit history changes commit IDs.

## Clean Untracked Files

```bash
git clean -fd                                   # Remove untracked files/folders
```

> **Warning:** git clean permanently deletes untracked files.

## Useful Flags

| Flag | Meaning |
|------|---------|
| -m | Commit message |
| -a | Include tracked modifications |
| -u | Set upstream branch |
| -d | Delete branch |
| -f | Force operation |
| --hard | Discard changes |
| --amend | Modify previous commit |
| --graph | Graph history |

## Common Errors

| Error | Cause | Fix |
|---------|--------|------|
| Merge conflict | Same lines edited | Resolve files then commit |
| Detached HEAD | Checked out commit directly | Switch back to branch |
| Non-fast-forward push | Remote ahead of local | Pull/rebase then push |
| Untracked file prevents checkout | File would be overwritten | Move or commit file |
| Permission denied | SSH/auth issue | Reconfigure credentials |

---

# Real-World Workflows

## Scenario 1: New Feature Development

```bash
git switch -c feature-login                     # Create feature branch
git add .                                       # Stage work
git commit -m "Implement login UI"             # Save progress
git push -u origin feature-login                # Publish branch
git switch main                                 # Return to main
git merge feature-login                         # Merge completed feature
```

## Scenario 2: Fix Production Bug

```bash
git switch main                                 # Move to main
git pull origin main                            # Get latest code
git switch -c hotfix-payment                    # Create hotfix branch
git commit -am "Fix payment timeout"           # Commit fix
git push -u origin hotfix-payment               # Push branch
git switch main                                 # Return to main
git merge hotfix-payment                        # Merge fix
```

## Scenario 3: Save Work Before Context Switch

```bash
git stash                                       # Store unfinished changes
git switch urgent-bug                           # Change task
git stash pop                                   # Restore saved work later
```

---

# Quick Reference

| Task | Command |
|------|---------|
| Initialize repository | git init |
| Clone repository | git clone URL |
| Check status | git status |
| Stage file | git add file |
| Stage everything | git add . |
| Commit changes | git commit -m "msg" |
| View log | git log |
| Create branch | git branch name |
| Switch branch | git switch name |
| Create and switch branch | git switch -c name |
| Merge branch | git merge branch |
| Push changes | git push origin branch |
| Pull updates | git pull origin branch |
| Fetch updates | git fetch |
| Stash work | git stash |
| Restore file | git restore file |
| Revert commit | git revert hash |
| Reset commit | git reset --soft HEAD~1 |
| Tag release | git tag v1.0.0 |
| Show remotes | git remote -v |
