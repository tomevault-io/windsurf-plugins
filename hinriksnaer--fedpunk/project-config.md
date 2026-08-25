---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fedpunk is a modular configuration management system for Fedora Linux, built entirely in Fish shell. It provides a minimal core (~500 KB) with external profile support for complete desktop environments based on Hyprland (Wayland compositor) with vim-style keybindings, live theming, and extensible module architecture.

**Core Philosophy:**
- Modular architecture with automatic dependency resolution
- Profile-based configuration (desktop/container/custom modes)
- GNU Stow for instant symlink-based deployment (no generation step)
- Fish-first shell experience with modern tooling
- External module support (git URLs, local paths, profile modules)

## Build and Test Commands

### Local Development

```fish
# Install Fedpunk from local checkout
fish install.fish

# Install with specific profile and mode
fish install.fish --profile default --mode desktop
fish install.fish --profile default --mode container

# Module management
fedpunk module list                    # List all modules
fedpunk module info <name>             # Show module details
fedpunk module deploy <name>           # Full deployment (deps + packages + config)
fedpunk module stow <name>             # Config only (symlink with stow)
fedpunk module unstow <name>           # Remove symlinks
fedpunk module install-packages <name> # Packages only
fedpunk module run-lifecycle <name> <hook>  # Run specific lifecycle hook
```

### Testing

```bash
# Build RPM package for local testing
bash test/build-rpm.sh

# Test RPM installation
bash test/ci/test-rpm-install.sh

# Run specific workflow tests locally (requires container runtime)
# See .github/workflows/ for available tests:
# - test-default-container.yml
# - test-default-desktop.yml
# - test-dev-desktop.yml
# - test-rpm-build.yml
```

### Installation Paths

Fedpunk supports two installation modes that are auto-detected:

1. **DNF/RPM installation** (COPR packages):
   - System files: `/usr/share/fedpunk/`
   - User data: `~/.local/share/fedpunk/`
   - Environment variables set via `/etc/profile.d/fedpunk.sh`

2. **Git clone installation** (traditional):
   - All files: `~/.local/share/fedpunk/`
   - Environment variables set by Fish during shell initialization

The `lib/fish/paths.fish` library automatically detects which mode is active and sets `FEDPUNK_SYSTEM`, `FEDPUNK_USER`, and `FEDPUNK_ROOT` accordingly.

## Architecture

### Module System

Every component (neovim, tmux, hyprland, etc.) is a self-contained module with:

**Directory Structure:**
```
modules/<package>/
├── module.yaml          # Metadata, dependencies, parameters, packages
├── config/              # Dotfiles (stowed to $HOME via symlinks)
│   └── .config/...
├── cli/                 # Optional CLI commands
│   └── <package>/
│       └── <package>.fish
└── scripts/             # Optional lifecycle hooks
    ├── install          # Custom installation logic
    ├── before           # Pre-deployment hook
    └── after            # Post-deployment hook (services, etc.)
```

**module.yaml schema:**
```yaml
module:
  name: fish
  description: Fish shell with modern tooling
  dependencies:
    - rust      # Modules required before this one
  priority: 10  # Execution order (lower = earlier)

parameters:          # Optional: define module parameters
  param_name:
    type: string
    description: Parameter description
    default: value
    required: false

environment:         # Environment variables (exported to shell)
  MY_VAR: "value"
  API_URL: "https://api.example.com"

lifecycle:
  before: []         # Hook names to run before stow
  after:
    - install        # Hook names to run after stow

packages:
  copr:              # COPR repositories
    - atim/starship
  dnf:               # DNF packages
    - fish
  cargo: []          # Cargo packages
  npm: []            # NPM packages
  flatpak: []        # Flatpak packages

stow:
  target: $HOME
  conflicts: warn    # warn, skip, or overwrite
```

### Deployment Flow

1. **Profile/Mode Selection** - Choose profile (default/dev/example) and mode (desktop/container)
2. **Module List Resolution** - Load module list from `profiles/<name>/modes/<mode>/mode.yaml`
3. **External Module Resolution** - Clone/cache git URLs, resolve local paths, locate profile modules
4. **Dependency Resolution** - Recursive topological sort, prevents duplicates
5. **Parameter Injection** - Generate Fish config for module parameters
6. **Environment Injection** - Generate Fish/Bash config for module environment variables
7. **Package Installation** - DNF, COPR, Cargo, NPM, Flatpak from module.yaml
8. **Lifecycle: before** - Pre-deployment hooks
9. **GNU Stow Deployment** - Symlink `config/` directories to `$HOME`
10. **Lifecycle: after** - Post-deployment hooks (services, etc.)

**Key Design Decision:** GNU Stow provides instant deployment via symlinks. Editing a file in a module's `config/` directory immediately affects the stowed location with no generation step.

### Profile System

**Profiles are external only.** Fedpunk core ships with no built-in profiles. Deploy profiles from git URLs:

```fish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hinriksnaer/fedpunk](https://github.com/hinriksnaer/fedpunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
