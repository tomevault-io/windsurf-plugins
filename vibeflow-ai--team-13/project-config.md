---
trigger: always_on
description: JJ Version Control System rules for safe Git-to-JJ migration within Cursor IDE and using JJ in workflow
---

# JJ Version Control System Rules for Cursor AI

## CRITICAL SAFETY PRINCIPLES
- NEVER use dangerous commands without explicit user request.
- ALWAYS provide safe alternatives to Git workflows.
- MAINTAIN Git compatibility when working in colocated repositories.
- PRESERVE existing Git history and branches during transition.
- EXPLAIN the differences between JJ and Git concepts clearly.

## JJ FUNDAMENTALS

### Core Concepts
- JJ uses "changes" instead of commits as the primary unit.
- Every change has a unique **Change ID** that persists across rewrites.
- The working copy is always a change (`@`), automatically tracked.
- No staging area – all modifications are automatically snapshotted.
- Conflicts are first-class objects that don't block workflow.
- **Bookmarks** (equivalent to Git branches) are pointers to changes.

### Safe Transition Strategy
1. Start with colocated repositories (`jj git init --colocate`).
2. Use JJ commands while maintaining Git compatibility.
3. Gradually adopt JJ-native workflows.
4. Keep Git remotes and tooling functional.

---

## ESSENTIAL JJ COMMANDS AND USAGE

### Repository Setup (Safe Git Integration)
```bash
# Initialize JJ in an existing Git repository (RECOMMENDED)
jj git init --colocate

# Clone with colocated setup
jj git clone --colocate <url>

# Create bookmark to track remote
jj bookmark track main@origin

# Safe configuration
jj config set --user user.name "Your Name"
jj config set --user user.email "your.email@example.com"
```

### Daily Workflow Commands
```bash
# Check repository status
jj status                   # Show working copy status
jj log                      # Show change history
jj log -r @                 # Show current change only

# Creating and managing changes
jj new                      # Create new empty change
jj new main                 # Create change from main bookmark
jj describe -m "message"    # Add description to current change
jj commit -m "message"      # Describe and create new change on top

# Working with bookmarks (Git branch equivalents)
jj bookmark list            # List all bookmarks
jj bookmark create name     # Create bookmark at current change
jj bookmark create name -r @ # Create bookmark at working copy
jj bookmark move name -r @  # Move bookmark to current change
jj bookmark delete name     # Delete bookmark

# Safe change navigation
jj edit <rev>               # Switch to different change
jj new @-                   # Create change on parent
jj prev                     # Move to parent change
jj next                     # Move to child change

# Change operations
jj show                     # Show current change details
jj show <rev>               # Show specific change
jj diff                     # Show changes in working copy
jj diff -r <rev>            # Show changes in specific change

# Safe change manipulation
jj squash                   # Squash working copy into parent
jj squash -r <a>::<b> -i    # Interactive squash between changes
jj duplicate <rev>          # Create copy of change
jj abandon <rev>            # Remove change (children preserved)

# File operations
jj restore <path>           # Restore file to parent version
jj restore --from <rev> <path>  # Restore from specific change
jj file track <path>        # Start tracking file
jj file untrack <path>      # Stop tracking file

# Conflict resolution
jj resolve                  # Resolve conflicts interactively
jj resolve --tool meld      # Use specific merge tool

# Safety and undo
jj undo                     # Undo last operation
jj op log                   # Show operation history
jj op restore <op_id>       # Restore to specific operation
```

### Git Integration Commands
```bash
# Syncing with Git remotes
jj git fetch                       # Fetch from all remotes
jj git fetch --remote origin       # Fetch from specific remote
jj git push                        # Push current bookmark
jj git push --bookmark <name>      # Push specific bookmark
jj git push --all-bookmarks        # Push all bookmarks
jj git push -c @-                  # Push parent change with auto-generated bookmark

# Git compatibility operations
jj git import                      # Import Git changes
jj git export                      # Export to Git
```

---

## WORKFLOW PATTERNS

### Basic Feature Development
```bash
# 1. Create feature branch
jj new main
jj bookmark create feature/awesome-feature

# 2. Make changes (files automatically tracked)
# Edit files...
jj describe -m "Add awesome feature"

# 3. Continue development
jj new            # Create new change for next iteration
# Edit more files...
jj describe -m "Improve awesome feature"

# 4. Push to remote
jj git push --bookmark feature/awesome-feature
```

### Updating from Remote
```bash
# Safe update pattern
jj git fetch
jj rebase -d main@origin  # Rebase current change stack
```

### Stack-based Development
```bash
# Create change stack
jj new main
jj describe -m "Foundation change"

jj new
jj describe -m "Build on foundation"

jj new
jj describe -m "Final touches"

# Push entire stack
jj bookmark create feature/stack -r @--  # Bookmark the bottom change
jj git push --bookmark feature/stack
```

### Conflict Resolution Workflow
```bash
# When conflicts occur

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VibeFlow-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
