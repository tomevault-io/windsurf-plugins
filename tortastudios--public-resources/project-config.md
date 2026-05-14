---
trigger: always_on
description: description: Enforces Sapling SCM as the default source control system for all agent conversations and development workflows
---

---
description: Enforces Sapling SCM as the default source control system for all agent conversations and development workflows
globs: "**/*"
alwaysApply: true
---

- **Core Philosophy: Trunk-Based Development with Local Bookmark Discipline**
  - **Human workflow**: Local Sapling bookmarks → push directly to main (no feature branches)
  - **Background agent workflow**: Feature branches → immediate merge → delete branch
  - **CRITICAL: One feature = one local bookmark** (Non-Negotiable - prevents mixing work locally)
  - **Zero stale branches**: Any feature branch should be merged and deleted immediately
  - **Smartlog is primary navigation**: Use `sl` to see your commit graph and navigate

- **CRITICAL: Repository Setup & .gitignore (MUST DO FIRST)**
  - **ALWAYS add .sl/ to .gitignore IMMEDIATELY** when setting up Sapling
  - **Never commit .sl/ metadata** - it causes massive conflicts (237+ files)
  - **Essential .gitignore entries for Sapling:**
    ```gitignore
    # Sapling SCM metadata (CRITICAL - prevents committing internal files)
    .sl/
    
    # Git metadata conflicts in Sapling workflow  
    .git/index
    .git/FETCH_HEAD
    .git/refs/
    .git/logs/
    ```
  - **If .sl/ already committed**: `git rm -r --cached .sl/` then commit the removal

- **Repository Initialization Best Practices**
  - **Fresh Sapling setup**: `sl init --git` in existing Git repo
  - **Configure remote immediately**: Add to `~/.config/sapling/sapling.conf`:
    ```ini
    [paths]
    default = https://github.com/your-org/your-repo.git
    ```
  - **First pull to import history**: `sl pull` to get Git commits into Sapling
  - **Test basic workflow**: Create bookmark, commit, push to verify setup

- **Daily Human Workflow (Trunk-Based with Sapling)**
  ```sh
  sl pull                   # Start with latest main
  sl                        # See your work (smartlog - primary navigation)
  sl book feature-name      # Create local bookmark for focused work
  sl status                 # Check file changes
  # Work on atomic changes...
  sl add <new-files>        # Add any new/untracked files (if needed)
  sl commit -m "atomic change"  # Commits all tracked changes (no staging needed)
  sl push --to main         # Push directly to main (trunk-based)
  sl goto main              # Return to main for next work
  ```

- **Key Daily Commands**
  - `sl` - Show smartlog (your primary navigation tool - most important command)
  - `sl status` - See changed files 
  - `sl book <name>` - Create local bookmark for focused work
  - `sl goto <name>` - Switch to bookmark or main
  - `sl add <files>` - Add new/untracked files to repository (required before first commit)
  - `sl commit -m "message"` - Commit all tracked changes (no staging needed for tracked files)
  - `sl push --to main` - Push directly to main (trunk-based development)
  - `sl absorb` - Auto-absorb pending changes into appropriate stack commits
  - `sl undo` - Undo last operation (simpler than Git)

- **Local Bookmark Discipline (Non-Negotiable)**
  - **Always create bookmark before starting work**: `sl book feature-name`
  - **One logical feature = one local bookmark** - Never mix unrelated work
  - **Do NOT pile multiple features into one bookmark** (Otherwise, airstrike consequences!)
  - **Check current bookmark before starting**: `sl bookmark` shows current with `*`
  - **Complete each feature before starting next** - Maintain atomic focus
  - **Return to main between features**: `sl goto main` after pushing

- **File Hygiene (Learned from Practice)**
  - **Always inspect before committing**: `sl status` then `sl diff`
  - **Use `file <filename>` to understand unfamiliar files** - Never commit unknown files
  - **Handle .gitignore conflicts**: `sl forget <file>` removes from tracking
  - **Git metadata in .git mode**: Add to .gitignore: `.git/index`, `.git/FETCH_HEAD`, `.git/refs/`, `.git/logs/`
  - **Remove accidentally tracked ignored files**: `sl forget <file>` then commit the removal

- **Status Code Interpretation (Critical Troubleshooting)**
  - **`M filename`** - Modified file, ready to commit
  - **`? filename`** - Untracked file, not in repository
  - **`! filename`** - Missing file (tracked but deleted/moved) - INVESTIGATE IMMEDIATELY
  - **Large list of `! .sl/` files** - Sapling metadata was accidentally committed, use recovery procedure

- **Emergency Recovery Procedures**
  - **If .sl/ metadata committed (causes `!` status everywhere)**:
    ```sh
    git rm -r --cached .sl/     # Remove from Git tracking
    echo ".sl/" >> .gitignore   # Prevent future commits
    git commit -m "fix: remove .sl/ metadata from tracking"
    ```
  - **If Sapling metadata corrupted**:
    ```sh
    cp -r .sl .sl-backup       # Backup if possible
    rm -rf .sl                 # Remove corrupted metadata
    sl init --git              # Reinitialize
    sl pull                    # Import Git history
    ```
  - **If configuration missing**: Edit `~/.config/sapling/sapling.conf` directly

- **Background Agent Branch Management**
  - **Cursor background agents create feature branches by default** (not under our control)
  - **Always merge and delete these branches immediately** after agent completion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tortastudios/public-resources](https://github.com/tortastudios/public-resources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
