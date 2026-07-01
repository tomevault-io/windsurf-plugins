---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Note**: For general development information, setup instructions, and contribution guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md). This file contains Claude Code-specific guidance and detailed architecture information.

## Project Overview

**ofsht** is a Git worktree management CLI tool written in Rust. It wraps `git worktree` commands and adds automation features like hooks, file copying, symlink creation, and zoxide integration.

## Development Commands

All canonical checks run inside the Nix dev shell. Plain `cargo` commands work for quick local iteration, but reproduce CI failures with `nix develop --command ...`.

### Essential Commands
```bash
# Run all quality checks before committing (required)
nix develop --command just check

# Build the project
nix develop --command cargo build
# or for release
nix develop --command cargo build --release

# Run all tests
nix develop --command cargo test
# or CI-equivalent
nix develop --command just test-ci

# Run specific test module
nix develop --command cargo test config::tests
nix develop --command cargo test zoxide::tests

# Check formatting
nix develop --command just fmt-ci

# Run clippy with strict warnings
nix develop --command just clippy-ci
```

### Testing the CLI
```bash
# Run the CLI in development
cargo run -- add feature-branch
cargo run -- ls
cargo run -- cd feature-branch
cargo run -- rm .
cargo run -- rm /path/to/worktree
cargo run -- rm feature-a feature-b  # Remove multiple worktrees
cargo run -- rm feature-a . feature-b  # Remove multiple including current

# Test open command (must be run inside tmux)
cargo run -- open            # Open all worktrees as windows
cargo run -- open --pane     # Open all worktrees as panes
cargo run -- open --window   # Explicitly use window mode

# Test tmux integration (must be run inside tmux)
cargo run -- add feature-branch --tmux

# Test sync command
cargo run -- sync
cargo run -- sync --link
cargo run -- sync --run --copy

# Test stdin input (auto-detected when piped; CLI arg always wins)
echo feature-branch | cargo run -- add
echo feature-branch | cargo run -- create
echo feature-branch | cargo run -- cd
printf 'feature-a\nfeature-b\n' | cargo run -- rm
cargo run -- add < /dev/null  # errors with "branch name required"

# Use release binary
./target/release/ofsht add feature-branch
```

See `docs/TEST.md` for comprehensive manual testing procedures.

### New Subcommand Documentation Checklist

When adding a new subcommand, update ALL of the following:
- `README.md`: ToC, Features section, new Usage section, Common Workflows
- `CONTRIBUTING.md`: Module Structure tree, Command Modules list
- `docs/TEST.md`: New verification section, Summary checklist
- `AGENTS.md`: Testing the CLI examples, Module Structure, Command Modules

## Code Architecture

### Module Structure

The codebase follows a modular design with clear separation of concerns:

```
src/
├── main.rs           # CLI entry point and command routing only (~160 lines)
├── cli.rs            # CLI definitions and completion logic
├── commands/         # Command handlers (extracted from main.rs)
│   ├── add.rs        # Add command with GitHub/tmux integration
│   ├── cd.rs         # Navigate to worktree
│   ├── common.rs     # Shared command utilities
│   ├── completion.rs # Generate shell completions
│   ├── create.rs     # Simple worktree creation
│   ├── init.rs       # Initialize config files
│   ├── list.rs       # List worktrees
│   ├── open.rs       # Open all worktrees in tmux
│   ├── rm.rs         # Remove worktrees
│   ├── shell_init.rs # Generate shell integration scripts
│   └── sync.rs       # Sync hooks to existing worktrees
├── config.rs         # TOML configuration loading (local + global)
├── domain/           # Domain models and logic
│   └── worktree.rs   # Worktree entry parsing and formatting
├── hooks.rs          # Hook execution engine (run/copy/link)
├── stdin.rs          # Stdin input helpers (TTY-aware) for piped argument resolution
├── integrations/     # External tool integrations
│   ├── fzf/          # Interactive selection
│   ├── gh/           # GitHub CLI integration
│   ├── git/          # Git operations abstraction
│   ├── tmux/         # Tmux window/pane creation
│   └── zoxide/       # Directory tracking
├── service/          # Business logic layer
│   └── worktree.rs   # Worktree creation service
└── color.rs          # Terminal color output
```

### Key Design Patterns

**Two-tier Configuration System** (`config/loader.rs`)
- Local config: `.ofsht.toml` in **main repository root** (highest priority)
  - **Always loaded from main repository root**, not from individual worktrees
  - Use `Config::load_from_repo_root(&repo_root)` for git operations
  - Ensures consistent behavior across all worktrees
  - **Integration settings are NOT read from local config** - they are always sourced from global config
- Global config: Respects `XDG_CONFIG_HOME` on all platforms
  - Uses `$XDG_CONFIG_HOME/ofsht/config.toml` if set (must be absolute path)
  - Fallbacks to `~/.config/ofsht/config.toml` otherwise

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wadackel/ofsht](https://github.com/wadackel/ofsht) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
