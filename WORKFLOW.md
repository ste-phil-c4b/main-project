# Git Submodule Workflow with GitHub Flow

This guide explains how to manage feature development when using git submodules with GitHub Flow.

## Table of Contents

1. [GitHub Flow Refresher](#github-flow-refresher)
2. [Submodule Basics](#submodule-basics)
3. [Scenario 1: Feature in Submodule Only](#scenario-1-feature-in-submodule-only)
4. [Scenario 2: Feature Spanning Main + Submodule](#scenario-2-feature-spanning-main--submodule)
5. [Scenario 3: Cloning and Working with Submodules](#scenario-3-cloning-and-working-with-submodules)
6. [Scenario 4: Branch Tracking for Feature Development](#scenario-4-branch-tracking-for-feature-development)
7. [Common Pitfalls](#common-pitfalls)
8. [Best Practices](#best-practices)

---

## GitHub Flow Refresher

GitHub Flow is a lightweight, branch-based workflow:

1. **Create a branch** from `main` for your feature
2. **Add commits** as you work
3. **Open a Pull Request** to discuss and review
4. **Merge** to `main` after approval
5. **Deploy** (main is always deployable)

With submodules, this workflow applies to **each repository independently**.

---

## Submodule Basics

### What is a Submodule?

A git submodule is a reference to a specific commit in another repository. The parent repository stores:
- The submodule's repository URL
- The specific commit SHA to use

### Key Concept: Commit References

**Important:** The parent repo tracks a specific commit, not a branch. When you update a submodule, you're changing which commit the parent points to.

```
main-project (parent)
├── .gitmodules          # URLs of submodules
└── libs/sub-lib-a       # Points to commit abc123 in sub-lib-a repo
```

---

## Scenario 1: Feature in Submodule Only

Use this when your feature only affects the submodule code.

### Step 1: Navigate to the Submodule

```bash
cd libs/sub-lib-a
```

### Step 2: Create a Feature Branch

```bash
# Make sure you're on main and up to date
git checkout main
git pull origin main

# Create your feature branch
git checkout -b feature/add-new-function
```

### Step 3: Make Changes and Commit

```bash
# Make your changes...
git add .
git commit -m "Add new function to library A"
```

### Step 4: Push and Create PR

```bash
git push -u origin feature/add-new-function
```

Then create a Pull Request on GitHub for the submodule repository.

### Step 5: After PR is Merged - Update Parent

Once the PR is merged to `main` in the submodule:

```bash
# Go back to parent project
cd ../..

# Update submodule to latest main
cd libs/sub-lib-a
git checkout main
git pull origin main
cd ../..

# Commit the submodule reference update in parent
git add libs/sub-lib-a
git commit -m "Update sub-lib-a to include new function"
git push origin main
```

---

## Scenario 2: Feature Spanning Main + Submodule

Use this when your feature requires changes in both the main project and a submodule.

### Step 1: Create Feature Branch in Submodule

```bash
cd libs/sub-lib-a
git checkout main
git pull origin main
git checkout -b feature/breaking-api-change
```

### Step 2: Create Corresponding Branch in Main Project

```bash
cd ../..
git checkout main
git pull origin main
git checkout -b feature/breaking-api-change
```

### Step 3: Develop in Parallel

Work on both codebases. The main project branch will reference your submodule's feature branch.

```bash
# In submodule - make library changes
cd libs/sub-lib-a
# ... make changes ...
git add .
git commit -m "Update API signature"
git push -u origin feature/breaking-api-change

# In main project - update code that uses the library
cd ../..
# ... make changes ...
git add .
git commit -m "Update usage of sub-lib-a new API"
```

### Step 4: Commit Submodule Reference in Main Branch

```bash
# Make sure submodule is on your feature branch commit
git add libs/sub-lib-a
git commit -m "Point to sub-lib-a feature branch for development"
git push -u origin feature/breaking-api-change
```

### Step 5: Merge Order (Critical!)

**Always merge the submodule PR first:**

1. Merge `feature/breaking-api-change` PR in `sub-lib-a` repo
2. Update main project's feature branch to point to merged commit:
   ```bash
   cd libs/sub-lib-a
   git checkout main
   git pull origin main
   cd ../..
   git add libs/sub-lib-a
   git commit -m "Update sub-lib-a to merged main"
   git push origin feature/breaking-api-change
   ```
3. Merge `feature/breaking-api-change` PR in `main-project` repo

---

## Scenario 3: Cloning and Working with Submodules

### Fresh Clone

```bash
# Clone with submodules in one command
git clone --recurse-submodules https://github.com/ste-phil-c4b/main-project.git
```

### Already Cloned Without Submodules

```bash
git submodule update --init --recursive
```

### Switching Branches with Submodule Changes

When you switch branches that have different submodule commits:

```bash
git checkout feature-branch
git submodule update --recursive
```

### Pulling Updates

```bash
git pull origin main
git submodule update --recursive
```

### Update Submodules to Latest Remote

```bash
# Update all submodules to their latest remote main
git submodule update --remote --merge
```

---

## Scenario 4: Branch Tracking for Feature Development

Use branch tracking when you want the parent project to easily stay in sync with a submodule's feature branch during active development.

### Understanding Branch Tracking

By default, submodules point to a specific commit SHA. With branch tracking, you configure a submodule to know which branch to follow, making it easier to fetch the latest commits.

**Important:** The parent still stores a commit SHA, not a dynamic branch reference. Branch tracking just makes `git submodule update --remote` fetch the latest from the configured branch.

### Step 1: Configure Branch Tracking

Edit `.gitmodules` to add a branch configuration:

```ini
[submodule "libs/sub-lib-a"]
    path = libs/sub-lib-a
    url = https://github.com/ste-phil-c4b/sub-lib-a.git
    branch = feature/my-feature
```

Or use the command:

```bash
git config -f .gitmodules submodule.libs/sub-lib-a.branch feature/my-feature
```

### Step 2: Update to Latest on Tracked Branch

```bash
# Update specific submodule to latest commit on its tracked branch
git submodule update --remote libs/sub-lib-a

# Or update all submodules to their tracked branches
git submodule update --remote
```

### Step 3: Commit the Updated Reference

```bash
# The submodule now points to the latest commit on the tracked branch
git add libs/sub-lib-a
git commit -m "Update sub-lib-a to latest on feature branch"
```

### Workflow: Parallel Development with Branch Tracking

This workflow is useful when actively developing in both repos simultaneously.

```bash
# 1. In submodule: create and push feature branch
cd libs/sub-lib-a
git checkout -b feature/new-api
# ... make changes ...
git commit -m "Add new API"
git push -u origin feature/new-api

# 2. In parent: create feature branch and configure tracking
cd ../..
git checkout -b feature/new-api
git config -f .gitmodules submodule.libs/sub-lib-a.branch feature/new-api
git add .gitmodules
git commit -m "Track sub-lib-a feature branch"

# 3. As submodule gets more commits, easily update parent
git submodule update --remote libs/sub-lib-a
git add libs/sub-lib-a
git commit -m "Update sub-lib-a to latest"

# 4. Before merging: switch back to main tracking
git config -f .gitmodules submodule.libs/sub-lib-a.branch main
# (after submodule PR is merged)
git submodule update --remote libs/sub-lib-a
git add .gitmodules libs/sub-lib-a
git commit -m "Switch sub-lib-a back to main branch"
```

### Switching Tracked Branches

```bash
# Change which branch the submodule tracks
git config -f .gitmodules submodule.libs/sub-lib-a.branch new-branch-name

# Fetch and update to the new branch
git submodule update --remote libs/sub-lib-a

# Commit both changes
git add .gitmodules libs/sub-lib-a
git commit -m "Switch sub-lib-a to track new-branch-name"
```

### When to Use Branch Tracking vs. Manual Updates

| Approach | Best For |
|----------|----------|
| **Branch Tracking** | Active parallel development, frequent submodule updates, team coordination on features |
| **Manual Updates** | Production releases, explicit control, infrequent updates, auditing requirements |

### Caveats

1. **Not Automatic:** You still need to run `git submodule update --remote` and commit the result
2. **PR Reviews:** Reviewers should check that `.gitmodules` branch changes are intentional
3. **CI/CD:** Build systems typically use `git submodule update` (not `--remote`), so they get the pinned commit
4. **Merge Conflicts:** Changing `.gitmodules` in multiple branches can cause merge conflicts

---

## Common Pitfalls

### 1. Forgetting to Update Submodule Reference

**Problem:** You merged changes in a submodule but didn't update the parent.

**Solution:** After merging submodule PRs, always:
```bash
cd libs/sub-lib-a
git checkout main && git pull
cd ../..
git add libs/sub-lib-a
git commit -m "Update sub-lib-a reference"
```

### 2. Detached HEAD in Submodule

**Problem:** After `git submodule update`, you're in detached HEAD state.

**Solution:** This is normal. To make changes, create or checkout a branch:
```bash
cd libs/sub-lib-a
git checkout main  # or your feature branch
```

### 3. Wrong Merge Order

**Problem:** Merged main project PR before submodule PR. Main now points to non-existent commit on main branch.

**Solution:** Always merge submodule PRs first. If you made this mistake:
1. Revert the main project merge or
2. Quickly merge the submodule PR

### 4. Submodule Not Updated After Pull

**Problem:** Pulled main project but submodule still shows old code.

**Solution:** Always run after pulling:
```bash
git submodule update --recursive
```

### 5. Accidentally Committing Submodule Changes

**Problem:** `git add .` in parent added unwanted submodule state.

**Solution:** Be explicit about what you commit:
```bash
git add specific-file.txt
# or reset the submodule
git checkout -- libs/sub-lib-a
```

---

## Best Practices

### 1. Use Descriptive Branch Names

Use the same branch name across repos for related changes:
```
main-project:   feature/user-authentication
sub-lib-a:      feature/user-authentication
```

### 2. Document Submodule Dependencies in PRs

In your main project PR description:
```markdown
## Dependencies
- Requires [sub-lib-a#42](link-to-pr) to be merged first
```

### 3. Consider Using git submodule foreach

Run commands across all submodules:
```bash
git submodule foreach 'git checkout main && git pull'
```

### 4. Set Up Aliases

```bash
git config alias.pull-all '!git pull && git submodule update --recursive'
git config alias.clone-all 'clone --recurse-submodules'
```

### 5. Review .gitmodules Changes

When PRs modify `.gitmodules`, review carefully:
- Is the URL correct?
- Is the path appropriate?
- Are there security implications?

### 6. Keep Submodules Focused

Each submodule should have a clear, single responsibility. This makes:
- Independent versioning easier
- Feature branches more focused
- Merge conflicts less likely

---

## Quick Reference

| Task | Command |
|------|---------|
| Clone with submodules | `git clone --recurse-submodules <url>` |
| Initialize submodules | `git submodule update --init --recursive` |
| Update to tracked commits | `git submodule update --recursive` |
| Update to latest remote | `git submodule update --remote --merge` |
| Check submodule status | `git submodule status` |
| Run command in all submodules | `git submodule foreach '<command>'` |
| Set tracked branch | `git config -f .gitmodules submodule.<path>.branch <branch>` |
| Update from tracked branch | `git submodule update --remote <path>` |

---

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Feature in Submodule Only                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  sub-lib-a repo:                                                │
│  main ─────●─────────────────●───────────────────               │
│             \               /                                    │
│              ●────●────●───● feature/x                          │
│              1    2    3   4 (PR merged)                        │
│                                                                  │
│  main-project repo:                                             │
│  main ─────────────────────────────●────                        │
│                                    5 (update submodule ref)     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                 Feature Spanning Both Repos                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  sub-lib-a repo:                                                │
│  main ─────●─────────────────●───────────────────               │
│             \               /                                    │
│              ●────●────●───● feature/y (merge FIRST)            │
│                                                                  │
│  main-project repo:                                             │
│  main ─────●────────────────────────────●────                   │
│             \                          /                         │
│              ●────●────●────●─────────● feature/y               │
│              (refs submodule          (update ref,              │
│               feature branch)          then merge)              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
