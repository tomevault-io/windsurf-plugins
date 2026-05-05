---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Swift CLI tool (`mcs`) that configures Claude Code with MCP servers, plugins, skills, hooks, and settings. Pure pack management engine with zero bundled content — all features come from external tech packs. Distributed via Homebrew.

## Commands

```bash
# Development
swift build                      # Build the CLI
swift test                       # Run tests
swift build -c release --arch arm64 --arch x86_64  # Universal binary

# CLI usage (after install)
mcs sync [path]                  # Sync project: multi-select packs, compose artifacts (default command)
mcs sync --pack ios              # Non-interactive: apply specific packs (repeatable)
mcs sync --all                   # Apply all registered packs without prompts
mcs sync --dry-run               # Preview what would change
mcs sync --customize             # Per-pack component selection
mcs sync --global                # Sync global scope (MCP servers, brew, plugins to ~/.claude/)
mcs sync --lock                  # Checkout locked versions from mcs.lock.yaml
mcs sync --update                # Fetch latest versions and update mcs.lock.yaml
mcs doctor                       # Diagnose installation health
mcs doctor --fix                 # Diagnose and auto-fix issues
mcs doctor --pack ios            # Only check a specific pack
mcs doctor --global              # Check globally-configured packs only
mcs pack add <source>            # Add a tech pack (git URL, GitHub shorthand, or local path)
mcs pack add user/repo           # GitHub shorthand → https://github.com/user/repo.git
mcs pack add /path/to/pack       # Add a local pack (read in-place, no clone)
mcs pack add <url> --ref <tag>   # Add at a specific tag, branch, or commit (git only)
mcs pack add <url> --preview     # Preview pack contents without installing
mcs pack remove <name>           # Remove an external tech pack
mcs pack remove <name> --force   # Remove without confirmation
mcs pack list                    # List registered external packs
mcs pack update [name]           # Update pack(s) to latest version (skips local packs)
mcs pack validate [source]       # Validate a tech pack (path, identifier, or current directory)
mcs cleanup                      # Find and delete backup files
mcs cleanup --force              # Delete backups without confirmation
mcs export <dir>                 # Export current config as a tech pack
mcs export <dir> --global        # Export global scope (~/.claude/)
mcs export <dir> --identifier id # Set pack identifier (prompted if omitted)
mcs export <dir> --non-interactive  # Include everything without prompts
mcs export <dir> --dry-run       # Preview what would be exported
mcs check-updates                # Check for pack and CLI updates
mcs check-updates --hook         # Run as SessionStart hook (24-hour cooldown, respects config)
mcs check-updates --json         # Machine-readable JSON output
mcs config list                  # Show all settings with current values
mcs config get <key>             # Get a specific setting value
mcs config set <key> <value>     # Set a configuration value (true/false)
```

## Architecture

### Swift Package Structure
- **Package.swift** — swift-tools-version: 6.0, macOS 13+, deps: swift-argument-parser, Yams
- **Sources/mcs/** — main executable target
- **Tests/MCSTests/** — test target

### Entry Point
- `CLI.swift` — `@main` struct, `MCSVersion.current`, subcommand registration (`SyncCommand` is the default subcommand)

### Core (`Sources/mcs/Core/`)
- `Constants.swift` — centralized string constants (file names, CLI paths, JSON keys, external packs, plugins)
- `Environment.swift` — paths, arch detection, brew path, claude-home cwd detection (`isInsideClaudeHome(_:)`)
- `CLIOutput.swift` — ANSI colors, logging, prompts, multi-select, doctor summary
- `ShellRunner.swift` — Process execution wrapper
- `Settings.swift` — Codable model for `settings.json` and `settings.local.json`, deep-merge
- `Backup.swift` — timestamped backups for mixed-ownership files (CLAUDE.local.md), backup discovery and deletion
- `GitignoreManager.swift` — global gitignore management, core entry list
- `ClaudeIntegration.swift` — `claude mcp add/remove` (with scope support), `claude plugin install/remove`
- `ClaudePrerequisite.swift` — Claude Code CLI availability check with optional Homebrew auto-install
- `Homebrew.swift` — brew detection, package install/uninstall
- `FileHasher.swift` — SHA-256 file and directory hashing via CryptoKit (used by `PackTrustManager` and `ComponentExecutor`)
- `FileLock.swift` — POSIX `flock()` process lock and `LockedCommand` protocol for mutually exclusive CLI commands
- `Lockfile.swift` — `mcs.lock.yaml` model for pinning pack commits
- `PathContainment.swift` — centralized path-boundary checks and relative-path utilities (symlink-safe containment, traversal prevention)
- `PluginRef.swift` — parsed `name@repo` plugin references with marketplace resolution
- `ProjectDetector.swift` — walk-up project root detection (`.git/` or `CLAUDE.local.md`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcs-cli/mcs](https://github.com/mcs-cli/mcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
