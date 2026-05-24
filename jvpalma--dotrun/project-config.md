---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DotRun (`dr`) is a unified Bash script management framework that provides instant access to custom scripts, aliases, and configurations from anywhere. It supports Bash, Zsh, and Fish shells.

**Version:** 3.1.3

## Development Setup

```bash
# Clone and set up development environment
./dev.sh

# Get help on setup script
./dev.sh --help
```

The `dev.sh` script sets up a complete development environment with 7 steps:

1. Symlinks `dr` binary to `~/.local/share/dotrun/`
2. Symlinks `.dr_config_loader` for shell integration
3. Symlinks core feature modules (`core/*.sh`) and `help-messages/` directory tree
4. Symlinks helper utilities (`helpers/*.sh`)
5. Symlinks VERSION file
6. Symlinks shell integration files for Bash, Zsh, and Fish (with stale symlink cleanup)
7. Creates `~/.local/bin` directory for binary access

This creates symlinks from `~/.local/share/dotrun` to `core/shared/dotrun/`, enabling live testing without reinstallation. The script validates required files (`core/shared/dotrun/dr` and `core/shared/dotrun/` structure) before proceeding.

**v3.1.3 updates:**

- Symlinks entire `core/help-messages/` directory tree (replaces old file-by-file copying)
- Cleans stale symlinks in shell directories before creating new ones (removes dangling links from deleted source files)

**Logging design:** The script uses color-coded output functions (`info`, `success`, `warning`, `error`) with unicode indicators for clear progress tracking during development setup.

## Common Commands

```bash
# Linting (ShellCheck is required)
shellcheck core/shared/dotrun/dr
find core/shared/dotrun -name "*.sh" -exec shellcheck {} +

# Test core workflows manually
dr set test-script    # Create script
dr test-script        # Run script
dr -L                 # List with docs
dr -a set test        # Create alias file
dr -c set test        # Create config file
dr -col list          # List collections
dr reload             # Reload shell config (applies alias/config changes)
dr upgrade --check    # Check for updates
dr upgrade            # Upgrade to latest version
```

## Architecture

### Directory Structure

```
core/
├── shared/dotrun/           # Core tool files (symlinked during dev/install)
│   ├── dr                   # Main executable (~45k LOC Bash)
│   ├── .dr_config_loader    # Shell config loader
│   ├── core/                # Feature modules
│   │   ├── aliases.sh       # Alias management system
│   │   ├── collections.sh   # Collection import/sync system
│   │   ├── config.sh        # Config/env var management
│   │   ├── upgrade.sh       # Self-update from GitHub releases
│   │   ├── templates/       # Script templates
│   │   └── help-messages/   # CLI help text by feature
│   │       ├── core/
│   │       │   ├── help-message.sh      # Main dr --help output
│   │       │   ├── no-command.sh        # dr (no command provided)
│   │       │   └── reload.sh            # dr -r / dr reload
│   │       ├── scripts/
│   │       │   ├── move.sh              # dr -s move (without args)
│   │       │   └── no-args.sh           # dr -s / dr scripts (no subcommand)
│   │       ├── aliases/
│   │       │   ├── init.sh              # dr -a init (initialization success)
│   │       │   ├── move.sh              # dr -a move (without args)
│   │       │   ├── no-args.sh           # dr -a / dr aliases (no subcommand)
│   │       │   └── set.sh               # dr -a set (without args)
│   │       ├── configs/
│   │       │   ├── move.sh              # dr -c move (without args)
│   │       │   ├── no-args.sh           # dr -c / dr config (no subcommand)
│   │       │   └── set.sh               # dr -c set (without args)
│   │       ├── collections/
│   │       │   ├── help-message.sh      # dr -col --help (detailed guide)
│   │       │   ├── no-args.sh           # dr -col (no subcommand)
│   │       │   ├── remove.sh            # dr -col remove (without args)
│   │       │   ├── init-success.sh      # dr -col init (success message)
│   │       │   ├── conflict-menu.sh     # Interactive conflict resolution menu
│   │       │   ├── import-menu.sh       # Resource selection menu (a/s/l/h/c/n)
│   │       │   ├── invalid-url.sh       # GitHub URL validation error
│   │       │   └── errors/              # Error handling scripts
│   │       │       ├── git-errors.sh    # Git operation errors (5 types)
│   │       │       └── file-errors.sh   # File operation errors (9 types)
│   │       ├── upgrade/
│   │       │   ├── no-args.sh           # dr upgrade (usage/help)
│   │       │   ├── check-result.sh      # dr upgrade --check (result display)
│   │       │   └── errors/              # Upgrade error handling scripts
│   │       │       └── network-errors.sh # Network errors (3 types)
│   │       └── helpers/
│   │           └── loadHelpers-usage.sh # loadHelpers function usage
│   ├── helpers/             # Shared helper functions
│   │   ├── constants.sh            # Shared constants (icons, colors, paths)
│   │   ├── list_feature_files_tree.sh  # Unified tree display for features
│   │   ├── lint.sh                 # Linting helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvPalma/dotrun](https://github.com/jvPalma/dotrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
