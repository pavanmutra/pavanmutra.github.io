# Version Control & Git - Interview Q&A

## Git Fundamentals

### Q1: Explain what Git is and how it differs from centralized version control.

**Answer:**
Git is a distributed version control system that tracks changes in files. Key differences from centralized systems (like SVN):

**Centralized (SVN):**
- Single central server
- Must be online to commit
- If server fails, everyone stops working
- Slower operations

**Distributed (Git):**
- Every developer has full copy of repository (including history)
- Can work offline
- If server fails, work continues locally
- Faster operations (local commits)
- Can have multiple remotes

**Benefits of Git:**
- Branching and merging is easy
- Fast performance
- Works offline
- Better collaboration
- Non-linear development (multiple branches)

### Q2: Explain the difference between Git and GitHub/GitLab.

**Answer:**
- **Git**: The version control software/tool itself. Runs on your computer. Manages your local repository.

- **GitHub/GitLab**: Web-based hosting services for Git repositories. They provide:
  - Remote storage for your repositories
  - Web interface to view code
  - Collaboration features (pull requests, issues)
  - CI/CD integration
  - Access control and permissions

**Analogy**: Git is the engine, GitHub/GitLab is the garage where you store your car.

You can use Git without GitHub/GitLab (just locally), but GitHub/GitLab make it easier to share and collaborate.

### Q3: Explain the Git workflow: add, commit, push, pull.

**Answer:**
**Basic workflow:**

1. **Working Directory**: Where you edit files
2. **Staging Area (Index)**: Files you've marked to be committed
3. **Local Repository**: Committed changes stored locally
4. **Remote Repository**: Shared repository (GitHub/GitLab)

**Commands:**

```bash
# 1. Make changes to files
echo "new code" > file.txt

# 2. Add files to staging area
git add file.txt
# or add all changes
git add .

# 3. Commit changes to local repository
git commit -m "Add new feature"

# 4. Push to remote repository
git push origin main

# 5. Pull latest changes from remote
git pull origin main
```

**Visual flow:**
```
Working Directory → git add → Staging Area → git commit → Local Repository → git push → Remote Repository
                                                                                    ↑
                                                                              git pull
```

### Q4: What is the difference between git fetch and git pull?

**Answer:**
- **git fetch**: Downloads changes from remote but doesn't merge them into your working directory. Safe operation - doesn't change your files.

```bash
git fetch origin
# Downloads changes but your files unchanged
# You can review changes before merging
```

- **git pull**: Downloads changes AND automatically merges them into your current branch. Combines `git fetch` + `git merge`.

```bash
git pull origin main
# Downloads and merges immediately
# May cause merge conflicts
```

**When to use:**
- **git fetch**: When you want to see what changed before merging
- **git pull**: When you're ready to update your branch immediately

**Best practice**: Use `git fetch` first to review, then merge manually if needed. `git pull` can surprise you with automatic merges.

### Q5: Explain Git branches and why they're important.

**Answer:**
A branch is a separate line of development. Think of it as creating a copy of your code to work on without affecting the main code.

**Why important:**
- **Isolation**: Work on features without breaking main code
- **Collaboration**: Multiple people can work on different features simultaneously
- **Experimentation**: Try new ideas without risk
- **Releases**: Maintain different versions (v1.0, v2.0)

**Common branches:**
- **main/master**: Production-ready code
- **develop**: Integration branch for features
- **feature/**: New features being developed
- **hotfix/**: Urgent fixes for production
- **release/**: Preparing for release

**Example:**
```bash
# Create and switch to new branch
git checkout -b feature/new-login

# Make changes and commit
git add .
git commit -m "Add new login feature"

# Switch back to main
git checkout main

# Merge feature branch
git merge feature/new-login
```

## Branching Strategies

### Q6: Explain GitFlow branching strategy.

**Answer:**
GitFlow is a branching model with specific branch types and purposes:

**Main branches:**
- **main/master**: Production code. Always stable, deployable.
- **develop**: Integration branch. Where features merge before release.

**Supporting branches:**
- **feature/**: New features. Branch from develop, merge back to develop.
- **release/**: Preparing new release. Branch from develop, merge to both develop and main.
- **hotfix/**: Urgent production fixes. Branch from main, merge to both main and develop.

**Workflow:**
```
main (production)
  ↑
  | hotfix/urgent-fix
  |
develop (integration)
  ↑
  | feature/login
  | feature/payment
  | release/v1.2
```

**Pros**: Clear structure, good for releases, separates concerns.
**Cons**: Complex, many branches, can be overkill for small teams.

### Q7: Explain GitHub Flow branching strategy.

**Answer:**
GitHub Flow is simpler than GitFlow - designed for continuous deployment:

**Branches:**
- **main**: Production code. Always deployable.
- **feature branches**: Any branch for new work.

**Workflow:**
1. Create feature branch from main
2. Make changes and commit
3. Push branch and create pull request
4. Review and discuss in pull request
5. Merge to main (after approval)
6. Deploy immediately

**Key principles:**
- Main branch is always deployable
- Feature branches have descriptive names
- Merge via pull requests
- Deploy immediately after merge

**Pros**: Simple, fast, good for continuous deployment, less overhead.
**Cons**: Less structure for complex release cycles.

**Example:**
```bash
# Create feature branch
git checkout -b add-user-profile

# Make changes, commit, push
git add .
git commit -m "Add user profile page"
git push origin add-user-profile

# Create pull request on GitHub
# After approval, merge to main
```

### Q8: What is trunk-based development?

**Answer:**
Trunk-based development means all developers work on a single branch (usually main/trunk). No long-lived feature branches.

**Approach:**
- Everyone commits directly to main (or very short-lived branches)
- Features are developed in small, incremental commits
- Use feature flags to hide incomplete features
- Continuous integration and deployment

**Workflow:**
```bash
# Developer 1
git checkout main
git pull
# Make small change
git commit -m "Add login button"
git push

# Developer 2
git checkout main
git pull
# Make small change
git commit -m "Add validation"
git push
```

**Benefits:**
- No merge conflicts (everyone on same branch)
- Fast feedback
- Continuous integration
- Simpler workflow

**Requirements:**
- Small, frequent commits
- Good test coverage
- Feature flags
- Strong CI/CD

**Best for**: Teams practicing continuous deployment, small teams, microservices.

### Q9: When would you use feature branches vs working directly on main?

**Answer:**
**Use feature branches when:**
- Feature takes multiple days/weeks to complete
- Multiple people working on same feature
- Need code review before merging
- Feature might be cancelled (easy to delete branch)
- Experimenting with risky changes
- Team uses pull request workflow

**Work directly on main when:**
- Small, quick changes (typos, small fixes)
- Team practices trunk-based development
- Changes are low risk
- You have strong test coverage
- Using feature flags for incomplete features

**Best practice**: Use feature branches for anything that takes more than a few hours or needs review. Small fixes can go directly to main if your team allows it.

## Merge vs Rebase

### Q10: Explain the difference between merge and rebase.

**Answer:**
Both combine changes from one branch into another, but differently:

**Merge:**
- Creates a merge commit that combines two branches
- Preserves complete history (you see when branches diverged and merged)
- History shows all branches

```bash
git checkout main
git merge feature-branch
# Creates merge commit
```

**History looks like:**
```
* Merge commit (combines main and feature)
|\
| * Feature commit 2
| * Feature commit 1
* |
* | Main commit 2
* | Main commit 1
```

**Rebase:**
- Replays your commits on top of target branch
- Creates linear history (no merge commits)
- Rewrites commit history

```bash
git checkout feature-branch
git rebase main
# Replays feature commits on top of main
```

**History looks like:**
```
* Feature commit 2 (replayed)
* Feature commit 1 (replayed)
* Main commit 2
* Main commit 1
```

**When to use:**
- **Merge**: When you want to preserve branch history, working with shared branches
- **Rebase**: When you want clean linear history, personal feature branches

**Never rebase shared/public branches** - it rewrites history and confuses others.

### Q11: What is a merge conflict and how do you resolve it?

**Answer:**
Merge conflict occurs when Git can't automatically combine changes from two branches (same lines changed differently).

**Example conflict:**
```bash
git merge feature-branch
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
```

**Conflict markers in file:**
```
<<<<<<< HEAD
Code from current branch (main)
=======
Code from branch being merged (feature-branch)
>>>>>>> feature-branch
```

**How to resolve:**
1. **Open conflicted file** in editor
2. **Choose which code to keep** (or combine both)
3. **Remove conflict markers** (<<<<<<, ======, >>>>>>)
4. **Stage resolved file**: `git add file.txt`
5. **Complete merge**: `git commit`

**Example resolution:**
```bash
# Before (with conflict)
<<<<<<< HEAD
function login() {
    console.log("Login");
}
=======
function login() {
    console.log("User login");
    validateUser();
}
>>>>>>> feature-branch

# After (resolved)
function login() {
    console.log("User login");
    validateUser();
}
```

**Tools to help:**
- `git mergetool`: Opens visual merge tool
- VS Code: Built-in conflict resolution
- `git status`: Shows which files have conflicts

**Prevention**: Pull frequently, communicate with team, keep branches short-lived.

### Q12: Explain interactive rebase and when to use it.

**Answer:**
Interactive rebase lets you modify commits before they're applied. Useful for cleaning up commit history.

**Common uses:**
- Combine multiple small commits into one
- Reorder commits
- Edit commit messages
- Remove commits
- Split commits

**Example:**
```bash
# Rebase last 3 commits
git rebase -i HEAD~3
```

**Opens editor with:**
```
pick abc123 Add login button
pick def456 Fix typo
pick ghi789 Add validation

# Commands:
# p, pick = use commit
# r, reword = edit commit message
# e, edit = modify commit
# s, squash = combine with previous commit
# d, drop = remove commit
```

**Common workflow:**
```bash
# Change to:
pick abc123 Add login button
squash def456 Fix typo
squash ghi789 Add validation

# This combines all 3 into one commit
```

**When to use:**
- Clean up messy commit history before merging
- Combine related commits
- Fix commit messages
- Remove accidental commits

**Warning**: Only use on commits that haven't been pushed to shared branches.

## Conflict Resolution

### Q13: How do you handle conflicts when multiple people work on the same file?

**Answer:**
**Prevention strategies:**
1. **Communicate**: Tell team what files you're working on
2. **Pull frequently**: `git pull` before starting work
3. **Small commits**: Commit and push often
4. **Divide work**: Assign different files/features to different people

**When conflict occurs:**
1. **Don't panic**: Conflicts are normal in team environments

2. **Pull latest changes**:
```bash
git pull origin main
# If conflict, Git will tell you
```

3. **Identify conflicted files**:
```bash
git status
# Shows files with conflicts
```

4. **Resolve conflicts**:
   - Open file, find conflict markers
   - Discuss with teammate if needed
   - Choose correct code (or combine)
   - Remove markers

5. **Test resolved code**: Make sure it works

6. **Complete merge**:
```bash
git add resolved-file.txt
git commit -m "Resolve merge conflict in file.txt"
```

**Tools:**
- **VS Code**: Visual conflict resolution
- **Git merge tool**: `git mergetool`
- **Three-way merge**: See both versions and common ancestor

**Best practice**: Resolve conflicts immediately, don't let them accumulate.

### Q14: Explain Git stash and when to use it.

**Answer:**
Git stash temporarily saves uncommitted changes so you can switch branches or pull updates.

**Common scenario:**
```bash
# You're working on feature branch
# Made changes but not ready to commit
# Need to switch to main to fix urgent bug

# Save current work
git stash
# or with message
git stash save "Work in progress on login feature"

# Switch branches (your changes are safe)
git checkout main

# Fix bug, commit, push
# ...

# Switch back to feature branch
git checkout feature-branch

# Restore your work
git stash pop
# or
git stash apply  # Keeps stash, use pop to remove
```

**Useful commands:**
```bash
git stash list          # See all stashes
git stash show          # See what's in stash
git stash drop          # Delete stash
git stash clear         # Delete all stashes
```

**When to use:**
- Need to switch branches but have uncommitted work
- Want to pull updates but have local changes
- Need clean working directory for testing
- Want to test something quickly

**Alternative**: Commit to feature branch (even if incomplete), then continue later.

## Best Practices

### Q15: What are Git best practices for commit messages?

**Answer:**
**Good commit message structure:**
```
Short summary (50 chars or less)

More detailed explanation if needed. Explain what and why,
not how. Wrap at 72 characters.

- Bullet points if needed
- One point per line
```

**Best practices:**
1. **Use imperative mood**: "Add feature" not "Added feature" or "Adds feature"
2. **Be specific**: "Fix login validation bug" not "Fix bug"
3. **Explain why**: If not obvious, explain why change was made
4. **Reference issues**: "Fix #123: Login validation error"
5. **Keep it short**: First line should be summary
6. **No period**: First line shouldn't end with period

**Examples:**

**Good:**
```
Add user authentication

Implement login and registration functionality with JWT tokens.
Includes password hashing and session management.

Fixes #45
```

**Bad:**
```
fix
```

```
Updated stuff
```

```
WIP - fixing things, need to test more, might break, but should work
```

### Q16: How do you undo changes in Git?

**Answer:**
**Different scenarios:**

1. **Undo uncommitted changes in working directory**:
```bash
# Discard changes to specific file
git checkout -- file.txt
# or (newer syntax)
git restore file.txt

# Discard all changes
git checkout -- .
# or
git restore .
```

2. **Unstage files** (remove from staging area):
```bash
git reset HEAD file.txt
# or
git restore --staged file.txt
```

3. **Undo last commit** (keep changes):
```bash
git reset --soft HEAD~1
# Changes remain staged
```

4. **Undo last commit** (unstage changes):
```bash
git reset HEAD~1
# or
git reset --mixed HEAD~1
# Changes remain in working directory
```

5. **Undo last commit** (discard changes):
```bash
git reset --hard HEAD~1
# ⚠️ WARNING: Loses all changes!
```

6. **Revert commit** (creates new commit that undoes changes):
```bash
git revert HEAD
# Safe for shared branches - doesn't rewrite history
```

**When to use:**
- **reset --soft**: Want to recommit with different message
- **reset --mixed**: Want to unstage and recommit
- **reset --hard**: Want to completely discard (use carefully!)
- **revert**: Undoing commits already pushed to shared branch

### Q17: Explain Git tags and their use.

**Answer:**
Tags mark specific points in history (usually releases). Like a bookmark that never moves.

**Types:**
- **Lightweight tags**: Just a pointer to a commit
- **Annotated tags**: Include metadata (author, date, message)

**Create tags:**
```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v1.0.0 abc123 -m "Release version 1.0.0"
```

**Use tags:**
```bash
# List tags
git tag

# Show tag details
git show v1.0.0

# Checkout tag
git checkout v1.0.0

# Push tags to remote
git push origin v1.0.0
# or push all tags
git push --tags
```

**Common uses:**
- Mark releases (v1.0.0, v2.1.3)
- Mark milestones
- Reference specific versions
- Create releases on GitHub/GitLab

**Best practice**: Use semantic versioning (v1.2.3) and annotated tags for releases.

### Q18: How do you find and fix mistakes in Git history?

**Answer:**
**Find mistakes:**

1. **View commit history**:
```bash
git log
git log --oneline
git log --graph --all
```

2. **Search commits**:
```bash
# Search commit messages
git log --grep="login"

# Search code changes
git log -S "functionName"

# Search by author
git log --author="John"
```

3. **View file history**:
```bash
git log file.txt
git blame file.txt  # See who changed each line
```

**Fix mistakes:**

1. **Amend last commit**:
```bash
# Forgot to add file
git add forgotten-file.txt
git commit --amend
# Updates last commit instead of creating new one
```

2. **Interactive rebase** (for older commits):
```bash
git rebase -i HEAD~5
# Edit, reword, or remove commits
```

3. **Revert commit** (safe for shared branches):
```bash
git revert abc123
# Creates new commit that undoes changes
```

4. **Reset** (only for local/unpushed commits):
```bash
git reset --hard abc123
# ⚠️ Only if not pushed to shared branch
```

**Best practice**: Use `revert` for shared branches, `reset`/`rebase` only for local branches.

### Q19: Explain Git hooks and their use cases.

**Answer:**
Git hooks are scripts that run automatically at certain points in Git workflow.

**Types of hooks:**
- **pre-commit**: Runs before commit. Can prevent commit if checks fail.
- **pre-push**: Runs before push. Can prevent push.
- **post-commit**: Runs after commit. Good for notifications.
- **pre-rebase**: Runs before rebase.

**Example pre-commit hook** (prevent commit if tests fail):
```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running tests..."
npm test

if [ $? -ne 0 ]; then
    echo "Tests failed! Commit aborted."
    exit 1
fi

echo "Running linter..."
npm run lint

if [ $? -ne 0 ]; then
    echo "Linter failed! Commit aborted."
    exit 1
fi
```

**Make executable:**
```bash
chmod +x .git/hooks/pre-commit
```

**Use cases:**
- Run tests before commit
- Check code style/linting
- Validate commit messages
- Prevent committing secrets
- Run security scans

**Tools**: Use frameworks like Husky (Node.js) or pre-commit (Python) to manage hooks easily.

### Q20: How do you handle large files in Git?

**Answer:**
Git is not good for large files (videos, large binaries, databases). They slow down repository.

**Solutions:**

1. **Git LFS (Large File Storage)**:
```bash
# Install Git LFS
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.mp4"

# Commit as normal
git add .gitattributes
git add large-file.psd
git commit -m "Add design file"
```

Git LFS stores large files separately, only downloads when needed.

2. **Exclude from Git**:
```bash
# Add to .gitignore
*.log
*.mp4
large-files/
```

3. **External storage**: Store large files in cloud storage (Azure Blob, S3), reference URLs in code.

4. **Separate repository**: Keep large files in separate repo, reference in main repo.

**Best practice**: Use Git LFS for binaries that need versioning, exclude temporary/large files that don't need versioning.

