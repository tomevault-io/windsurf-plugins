---
trigger: always_on
description: When a task involves multiple steps (e.g., implement + commit + PR), complete ALL steps in sequence without stopping. If creating a branch, committing, and opening a PR, finish the entire chain.
---

# CLAUDE.md

## General Workflow

When a task involves multiple steps (e.g., implement + commit + PR), complete ALL steps in sequence without stopping. If creating a branch, committing, and opening a PR, finish the entire chain.

Always commit after every turn. Don't wait for the user to ask — if you made changes, commit them before responding. Do not ask "shall I commit?" or "want me to commit?" — just commit. Committing is not a destructive or risky action; it is the expected default after every change.

PR descriptions should be concise and changelog-oriented: what changed, why, and how to use it. Do not include test plans, design decisions, or implementation details — those belong in specs and commit messages.

## Project Overview

msgvault is an offline Gmail archive tool that exports and stores email data locally with full-text search capabilities. The goal is to archive 20+ years of Gmail data from multiple accounts, make it searchable, and eventually delete emails from Gmail once safely archived.

## Architecture (Go)

Single-binary Go application:

```
msgvault/
├── cmd/msgvault/            # CLI entrypoint
│   └── cmd/                 # Cobra commands
├── internal/                # Core packages
│   ├── tui/                 # Bubble Tea TUI
│   ├── query/               # DuckDB query engine over Parquet
│   ├── store/               # SQLite database access
│   ├── deletion/            # Deletion staging and manifest
│   ├── gmail/               # Gmail API client
│   ├── sync/                # Sync orchestration
│   ├── oauth/               # OAuth2 flows (browser + device)
│   └── mime/                # MIME parsing
│
├── go.mod                   # Go module
└── Makefile                 # Build targets
```

## Quick Commands

```bash
# Build
make build                    # Debug build
make build-release            # Release build (optimized)
make install                  # Install to ~/.local/bin or GOPATH
make test                     # Run tests
make lint                     # Run linter

# CLI usage
./msgvault init-db                                    # Initialize database
./msgvault add-account you@gmail.com                  # Browser OAuth
./msgvault add-account you@gmail.com --headless       # Device flow
./msgvault add-account you@acme.com --oauth-app acme  # Named OAuth app
./msgvault sync-full you@gmail.com --limit 100        # Sync with limit
./msgvault sync-full you@gmail.com --after 2024-01-01 # Sync date range
./msgvault sync-incremental you@gmail.com             # Incremental sync

# TUI and analytics
./msgvault tui                                        # Launch TUI
./msgvault tui --account you@gmail.com                # Filter by account
./msgvault tui --local                                # Force local (override remote config)
./msgvault build-cache                                # Build Parquet cache
./msgvault build-cache --full-rebuild                 # Full rebuild
./msgvault stats                                      # Show archive stats

# Apple Mail import
./msgvault import-emlx                                # Auto-discover accounts
./msgvault import-emlx ~/Library/Mail                 # Explicit mail directory
./msgvault import-emlx --account me@gmail.com         # Specific account(s)
./msgvault import-emlx /path/to/dir --identifier me@gmail.com  # Manual fallback

# Daemon mode (NAS/server deployment)
./msgvault serve                                      # Start HTTP API + scheduled syncs

# Maintenance
./msgvault repair-encoding                            # Fix UTF-8 encoding issues
```

## Key Files

### CLI (`cmd/msgvault/cmd/`)
- `root.go` - Cobra root command, config loading
- `syncfull.go` - Full sync command implementation
- `syncincremental.go` - Incremental sync command
- `tui.go` - TUI command, cache auto-build
- `build_cache.go` - Parquet cache builder (DuckDB)
- `repair_encoding.go` - UTF-8 encoding repair

- `import_emlx.go` - Apple Mail .emlx import command

### Core (`internal/`)
- `tui/model.go` - Bubble Tea TUI model and update logic
- `tui/view.go` - View rendering with lipgloss styling
- `query/engine.go` - DuckDB query engine over Parquet files
- `query/sqlite.go` - SQLite query engine (fallback)
- `store/store.go` - SQLite database operations
- `store/schema.sql` - Core SQLite schema
- `store/schema_sqlite.sql` - FTS5 virtual table
- `deletion/manifest.go` - Deletion staging and manifest generation
- `gmail/client.go` - Gmail API client with rate limiting
- `oauth/oauth.go` - OAuth2 flows (browser + device)
- `sync/sync.go` - Sync orchestration, MIME parsing
- `mime/parse.go` - MIME message parsing

### TUI Keybindings
- `j/k` or `↑/↓` - Navigate rows
- `Enter` - Drill down into selection
- `Esc` or `Backspace` - Go back
- `Tab` - Cycle views (Senders → Sender Names → Recipients → Recipient Names → Domains → Labels → Time)
- `s` - Cycle sort field (Name → Count → Size)
- `r` - Reverse sort direction
- `t` - Jump to Time view (cycle granularity when already in Time)
- `a` - Filter by account
- `f` - Filter by attachments
- `Space` - Toggle selection
- `A` - Select all visible
- `x` - Clear selection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wesm/msgvault](https://github.com/wesm/msgvault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
