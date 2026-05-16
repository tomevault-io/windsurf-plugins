---
trigger: always_on
description: This guide provides comprehensive instructions for Claude Code agents working with team members on development projects. It synthesizes best practices, workflows, and critical guidelines to ensure effective and safe code contributions.
---

# Claude Code Agent Guide for Ithaca Team

This guide provides comprehensive instructions for Claude Code agents working with team members on development projects. It synthesizes best practices, workflows, and critical guidelines to ensure effective and safe code contributions.

## Directory Structure and Purpose

Your projects directory contains various git repositories for reference and development. Each subdirectory is typically an independent git repository used for finding code examples, implementations, and reference material.

**CRITICAL**: Reference repositories are for REFERENCE ONLY. Do not modify git configurations or remotes in these repositories.

## Essential Workflow Rules

### 1. Working Directory Guidelines

- **Read/explore**: Your main projects directory (reference repositories)
- **Modify/experiment**: A dedicated workspace directory for isolated changes

**ALWAYS check if a repository is already cloned locally before attempting to fetch from the web!** Use `ls` or check the directory structure to see if the project you need is already present before cloning it for reference.

### 2. Code Modification Workflow

When working on code changes:

1. **Always work in a dedicated workspace directory**
2. **Use git worktrees** for creating isolated workspaces from existing repos
3. **Only use git clone if the repository doesn't exist locally**
4. **NEVER modify the remotes of existing reference repositories**

#### Git Worktree Workflow (Preferred)

Git worktrees allow multiple working directories from a single repository, perfect for parallel work:

```bash
# First, check if the repo exists in your workspace
cd ~/projects  # or your designated workspace directory
ls -la | grep REPO_NAME

# If repo exists, create a worktree
cd ~/projects/REPO_NAME
git worktree add ../REPO_NAME-FEATURE-PURPOSE -b feature-branch

# If repo doesn't exist, clone it first (check for your fork)
gh repo view YOUR_USERNAME/REPO_NAME --web 2>/dev/null || echo "No fork found"

# Clone from your fork if it exists
git clone git@github.com:YOUR_USERNAME/REPO_NAME.git REPO_NAME
cd REPO_NAME
git remote add upstream git@github.com:ORIGINAL_ORG/REPO_NAME.git

# Or clone from original if no fork
git clone git@github.com:ORIGINAL_ORG/REPO_NAME.git REPO_NAME
```

#### Worktree Management

```bash
# List all worktrees for a repo
git worktree list

# Create a new worktree for a feature
git worktree add ../repo-optimization -b optimize-feature

# Remove a worktree when done
git worktree remove ../repo-optimization

# Clean up worktree references
git worktree prune
```

Use descriptive worktree names that indicate purpose:
- `reth-cpu-optimization`
- `repo-name-issue-123`
- `project-feature-description`

**Benefits of worktrees**:
- Share the same git history and objects (saves disk space)
- Switch between features instantly without stashing
- Keep multiple experiments running in parallel
- Easier cleanup - just remove the worktree directory

#### Workspace Maintenance

**Clean up build artifacts** when disk space is needed:
```bash
# For Rust projects
cd ~/projects/repo-name
cargo clean

# Check space used by all target directories
du -sh ~/projects/*/target

# Clean all Rust projects in workspaces - only run this if you are sure you want to clean all workspaces and/or have already run out of space
find ~/projects -name "Cargo.toml" -exec dirname {} \; | xargs -I {} sh -c 'cd {} && cargo clean'
```

**When to clean up workspaces**:
- After PR has been merged
- When changes have been abandoned
- Before removing a worktree

```bash
# Clean and remove a worktree
cd ~/projects/repo-name
git worktree remove ../repo-optimization
```

### 3. Git Workflow

When working on code:

1. Create feature branches for your work
2. Commit changes with clear messages
3. Use descriptive branch names: `name/fix-something`, `name/add-feature`, `name/make-clippy-happy-123456`

### 4. GitHub CLI (gh) Usage

The `gh` CLI tool is available for exploring GitHub repositories and understanding code context.

#### Allowed Operations

```bash
# Repository exploration
gh repo view owner/repo
gh repo clone owner/repo  # For initial clones

# Issues and PRs
gh issue list --repo owner/repo
gh issue view 123 --repo owner/repo
gh pr list --repo owner/repo
gh pr view 456 --repo owner/repo
gh pr diff 456 --repo owner/repo
gh pr checkout 456  # To examine PR branches locally

# API queries
gh api repos/owner/repo/pulls/123/comments
gh api repos/owner/repo/issues/123/comments

# Search operations
gh search issues "query" --repo owner/repo
gh search prs "query" --repo owner/repo

# Status and authentication
gh auth status
gh status

# Releases and workflows (read-only)
gh release list --repo owner/repo
gh release view v1.0.0 --repo owner/repo
gh workflow list --repo owner/repo
gh run list --workflow=ci.yml --repo owner/repo
```

#### Common Use Cases

1. **Examining PR discussions**:
   ```bash
   gh pr view 123 --comments
   gh api repos/paradigmxyz/reth/pulls/123/comments | jq '.[].body'
   ```

2. **Finding related issues**:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ithacaxyz/claude-md](https://github.com/ithacaxyz/claude-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
