---
trigger: always_on
description: Provides:
---

# CLAUDE.md

Project-specific guidance for Claude Code when working with this repository.

## Project Overview

Sindri is a complete AI-powered cloud development forge running on Fly.io infrastructure. It provides cost-optimized,
secure virtual machines with persistent storage for AI-assisted development without requiring local installation.
Like the legendary Norse blacksmith, Sindri forges powerful development environments from cloud infrastructure,
AI tools, and developer workflows.

## Development Commands

### VM Management

```bash
./scripts/vm-setup.sh --app-name <name>  # Deploy new VM
./scripts/vm-suspend.sh                  # Suspend to save costs
./scripts/vm-resume.sh                   # Resume VM
./scripts/vm-teardown.sh                 # Remove VM and volumes
flyctl status -a <app-name>             # Check VM status

# CI/Testing deployment (disables SSH daemon, health checks)
CI_MODE=true ./scripts/vm-setup.sh --app-name <test-name>
flyctl deploy --strategy immediate --wait-timeout 60s  # Skip health checks
```

### On-VM Commands

```bash
extension-manager list                            # List available extensions
extension-manager --interactive                   # Interactive extension setup
extension-manager install <name>                  # Install specific extension
extension-manager install-all                     # Install all active extensions
claude                                            # Authenticate Claude Code
npx claude-flow@alpha init --force               # Initialize Claude Flow in project
new-project <name> [--type <type>]               # Create new project with enhancements
clone-project <url> [options]                    # Clone and enhance repository
```

## Key Directories

### User Workspace (developer-owned, writable)

- `/workspace/developer/` - Developer home directory (persistent)
- `/workspace/scripts/` - User scripts and extension-generated helpers
- `/workspace/projects/active/` - Active development projects
- `/workspace/config/` - User configuration files
- `/workspace/bin/` - User binaries (in PATH)
- `/workspace/docs/` - Workspace-wide documentation

### System Directories (managed by Sindri)

- `/docker/lib/` - System libraries and extension definitions (immutable, from Docker image)
- `/docker/lib/extensions.d/` - Extension definitions and templates
- `/workspace/.system/` - System runtime files (do not modify directly)
  - `bin/` - System binaries (symlinked to /docker)
  - `lib/` - System libraries (symlinked to /docker)
  - `manifest/` - Extension activation configuration

All user data in `/workspace` persists between VM restarts. System files in `/docker` are immutable
and updated only via Docker image rebuilds. The `.system/` directory uses symlinks for efficiency.

## Development Workflow

### Daily Tasks

1. Connect via SSH: `ssh developer@<app-name>.fly.dev -p 10022`
   - Alternative: `flyctl ssh console -a <app-name>` (uses Fly.io's hallpass service)
2. Work in `/workspace/` (all data persists)
3. VM auto-suspends when idle
4. VM auto-resumes on next connection

### Project Creation

```bash
# New project
new-project my-app --type node

# Clone existing
clone-project https://github.com/user/repo --feature my-feature

# Both automatically:
# - Create CLAUDE.md context
# - Initialize Claude Flow
# - Install dependencies
```

## Extension System (v1.0)

Sindri uses a manifest-based extension system to manage development tools and environments.

### Extension Management

```bash
# List all available extensions
extension-manager list

# Interactive setup with prompts (recommended for first-time setup)
extension-manager --interactive

# Install an extension (auto-activates if needed)
extension-manager install <name>

# Install all active extensions from manifest
extension-manager install-all

# Check extension status
extension-manager status <name>

# Validate extension installation
extension-manager validate <name>

# Validate all installed extensions
extension-manager validate-all

# Uninstall extension
extension-manager uninstall <name>

# Reorder extension priority
extension-manager reorder <name> <position>

# Upgrade commands (Extension API v2.0)
extension-manager upgrade <name>         # Upgrade specific extension
extension-manager upgrade-all            # Upgrade all extensions
extension-manager upgrade-all --dry-run  # Preview upgrades
extension-manager check-updates          # Check for updates
extension-manager upgrade-history        # View upgrade history
```

### Automatic Dependency Resolution (Extension API v2.2)

Extensions automatically install their dependencies without manual intervention:

```bash
# Example: Install openskills (requires nodejs + git)
extension-manager install openskills
# → Auto-installs nodejs first
# → Auto-installs git
# → Installs openskills

# Show dependency tree without installing
extension-manager resolve playwright
# Output: nodejs, playwright
```

**Dependency Graph:**

- `openskills` → nodejs, git
- `monitoring` → python
- `playwright` → nodejs
- `nodejs-devtools` → nodejs

**How It Works:**

1. System detects `EXT_DEPENDENCIES` metadata from extension files
2. Builds dependency graph with cycle detection
3. Performs topological sort to determine install order
4. Adds all dependencies to manifest in correct order

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pacphi/sindri-legacy](https://github.com/pacphi/sindri-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
