---
trigger: always_on
description: <!-- NOTE: This file must stay in sync with CLAUDE.md. If you update one, update the other. -->
---

<!-- NOTE: This file must stay in sync with CLAUDE.md. If you update one, update the other. -->

## Project Overview

ComfyGit is a monorepo workspace using uv for Python package management. It provides unified environment management for ComfyUI through multiple coordinated packages. Licensed under GPL-3.0.

### Packages

| Package | PyPI Name | CLI Commands | Description |
|---------|-----------|--------------|-------------|
| `packages/core/` | comfygit-core | — | Core library for environment management |
| `packages/cli/` | comfygit | `comfygit`, `cg` | Main CLI |
| `packages/deploy/` | comfygit-deploy | `cg-deploy` | Remote deployment |

### Codebase Navigation

**For implementation tasks**, read the relevant architecture overview first:
- @packages/core/docs/architecture.md - Core library: layers, managers, services, protocols
- @packages/cli/docs/architecture.md - CLI: command handlers, strategies, formatters
- @packages/deploy/docs/architecture.md - Deploy: providers, worker server, async patterns

**Then use pyast** for specific symbol lookups:
```bash
# overview/symbols take a DIRECTORY (not a file)
pyast overview packages/core/src/comfygit_core/
pyast symbols packages/core/src/comfygit_core/utils/

# search takes PATH first, then QUERY (pyast search <path> <query>)
pyast search packages/core/src/comfygit_core/ "retry"
pyast search packages/core/src/comfygit_core/managers/ "injection_context"

# deps takes SYMBOL then FILE
pyast deps "Environment.sync" packages/core/src/comfygit_core/core/environment.py
```

**Key utility locations** (check before reimplementing):
- `packages/core/src/comfygit_core/utils/` - git, filesystem, retry, parsing helpers
- `packages/core/src/comfygit_core/services/` - downloads, lookups, registry

## CLI Reference

### Global Commands
```bash
cg init                          # Initialize workspace
cg list                          # List all environments
cg update                        # Upgrade ComfyGit CLI to latest version
cg update --check                # Check for updates without upgrading
cg import <source>               # Import environment from tarball or git URL
cg export                        # Export environment
cg config                        # Manage configuration settings
cg completion                    # Manage shell tab completion
```

### Environment Lifecycle
```bash
cg create <name>                 # Create new environment
cg use <name>                    # Set active environment
cg delete <name>                 # Delete environment
cg -e <name> run                 # Run ComfyUI
cg -e <name> run --no-sync       # Run without syncing first
cg -e <name> run -- --port 8189  # Pass args to ComfyUI via --
cg -e <name> status              # Show sync + git status
cg -e <name> manifest            # Show pyproject.toml contents
cg -e <name> sync                # Sync packages and dependencies
cg -e <name> repair              # Repair environment to match pyproject
cg -e <name> doctor              # Diagnose and repair uv tooling
cg -e <name> doctor --check-only # Check only, don't repair
```

### Node Management
```bash
cg -e <name> node add <id>           # Add node (registry ID, GitHub URL, or local dir)
cg -e <name> node add <id> --dev     # Track existing local development node
cg -e <name> node add <id> --strict  # Fail on dependency conflicts
cg -e <name> node remove <id>        # Remove node
cg -e <name> node update <id>        # Update node to latest
cg -e <name> node list               # List installed nodes
```

### Python Dependencies
```bash
cg -e <name> py add <packages>              # Add Python dependencies
cg -e <name> py add <pkg> --no-build-isolation  # For CUDA packages needing PyTorch at build time
cg -e <name> py add <pkg> --optional <extra> # Add to optional dependency group
cg -e <name> py add <pkg> --group <group>    # Add to dependency group
cg -e <name> py add <pkg> --dev              # Add to dev dependencies
cg -e <name> py add <pkg> --editable         # Install as editable
cg -e <name> py add -r requirements.txt      # Add from requirements file
cg -e <name> py remove <packages>            # Remove dependencies
cg -e <name> py remove-group <group>         # Remove entire dependency group
cg -e <name> py list                         # List dependencies
cg -e <name> py uv <args>                    # Direct UV passthrough (advanced)
```

### Git Operations
```bash
cg -e <name> commit -m "message"  # Commit environment changes
cg -e <name> log                  # Show commit history
cg -e <name> branch               # List/create/delete branches
cg -e <name> switch <branch>      # Switch branches
cg -e <name> checkout <ref>       # Checkout commits/branches/files
cg -e <name> pull                 # Pull and repair environment
cg -e <name> push                 # Push to remote
cg -e <name> remote               # Manage git remotes
cg -e <name> merge <branch>       # Merge branch
cg -e <name> reset                # Reset HEAD
cg -e <name> revert               # Revert a commit
```

### Environment Configuration
```bash
# PyTorch backend (machine-specific, gitignored)
cg -e <name> env-config torch-backend show     # Show current backend

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [comfygit-ai/comfygit](https://github.com/comfygit-ai/comfygit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
