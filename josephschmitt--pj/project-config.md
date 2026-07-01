---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`pj` is a fast project directory finder CLI written in Go that searches filesystems for git repositories and project directories. It's designed for speed and seamless integration with fuzzy finders like `fzf`.

## Development Environment

This project uses [devbox](https://get.jetify.com/devbox) to manage development tools (Go, golangci-lint, lefthook, etc.). Run `devbox shell` to enter the development environment, or use [direnv](https://direnv.net/) for automatic activation.

## Build Commands

All commands must be run inside the devbox environment. For common tasks, prefer `devbox run <script>`. For other make targets, use `devbox run -- make <target>`.

```bash
# Build for current platform
devbox run build

# Run tests with coverage
devbox run test

# Run linter
devbox run lint

# Run tests with race detector and detailed coverage
devbox run -- make test-coverage

# Install to $GOPATH/bin
devbox run -- make install

# Run the binary
devbox run -- make run

# Cross-compile for all platforms (outputs to dist/)
devbox run -- make build-all

# Validate GoReleaser config
devbox run -- make release-check

# Test GoReleaser locally without publishing
devbox run -- make release-local

# Clean build artifacts
devbox run -- make clean
```

## Architecture

### Core Data Flow

1. **main.go** - CLI entry point using `kong` for argument parsing
2. **config package** - Loads YAML config from `~/.config/pj/config.yaml` or XDG_CONFIG_HOME, merges with CLI flags
3. **cache package** - Manages JSON cache files in `~/.cache/pj/` or XDG_CACHE_HOME with TTL-based invalidation
4. **discover package** - Walks filesystem concurrently to find project directories
5. **icons package** - Maps project markers (`.git`, `go.mod`, etc.) to Nerd Font icons and ANSI colors

### Key Design Patterns

**Concurrent Discovery**: The `discover` package uses a fan-out goroutine pattern - one goroutine per search path, all feeding results into a shared channel. This provides significant speedup when searching multiple root directories.

**Config-Based Cache Keys**: Cache files are named using a SHA256 hash of the configuration (search paths, markers, excludes, max depth). This means different configurations automatically get separate cache files, preventing stale results when settings change.

**Priority-Based Sorting**: Projects are sorted by marker specificity (e.g., `package.json` has priority 10, `.git` has priority 1), then alphabetically. This ensures language-specific projects appear before generic git repos.

**Early Termination**: When a project marker is found, that directory subtree is skipped (returns `fs.SkipDir`). This prevents redundant scanning and respects that a parent project marker takes precedence over child markers.

**Worktree Detection**: Git worktrees are detected via two paths: Path A detects `.git` files (vs directories) during the normal walk and parses the `gitdir` reference to identify the parent repo. Path B (enabled with `--worktrees`) reads `.git/worktrees/` entries from parent repos to discover linked worktrees outside search paths. `--no-worktrees` filters worktrees from results. Worktree metadata (`IsWorktree`, `WorktreeParent`) is exposed in JSON output, format strings (`%w`), and display labels.

## Configuration System

The config loading order is:
1. Defaults (defined in `config.defaults()`)
2. YAML file (`~/.config/pj/config.yaml`)
3. CLI flags (highest priority)

CLI flags use reflection to merge into config struct, avoiding tight coupling between CLI and config packages.

### ANSI Color System

The `--ansi` flag wraps icons in ANSI foreground color codes (`\033[<code>m<icon>\033[39m`). Colors are configured per-marker via the `color` field in `MarkerConfig` (default: "blue") and can be overridden with `--color-map MARKER:COLOR`. The `icons.Mapper.Format()` method handles ANSI wrapping. Supported colors: black, red, green, yellow, blue, magenta, cyan, white, and bright- variants (16 total).

## Release Process

This project uses **Conventional Commits** for automated releases:

```bash
# Commit format
feat: new feature      # Minor version bump (0.1.0 → 0.2.0)
fix: bug fix          # Patch version bump (0.1.0 → 0.1.1)
feat!: breaking       # Major version bump (0.1.0 → 1.0.0)
docs: documentation   # No version bump
```

**Automated release workflow:**
1. Push commits with conventional format to `main`
2. `release-please` analyzes commits and creates/updates a Release PR
3. Merge the Release PR to trigger tag creation
4. Tag triggers GoReleaser to build and publish:
   - GitHub release with binaries for 6 platforms (darwin/linux/windows × amd64/arm64)
   - Homebrew formula to `josephschmitt/tap`
   - Scoop manifest to `josephschmitt/scoop-bucket`
   - Linux packages (.deb, .rpm, .apk, .pkg.tar.zst)

**Do not manually create tags.** The release-please workflow handles version management.

### Commit Guidelines

**Keep commits focused and atomic:**
- Each commit should address a single concern or change
- Never mix unrelated changes (e.g., bug fixes + new features) in one commit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josephschmitt/pj](https://github.com/josephschmitt/pj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
