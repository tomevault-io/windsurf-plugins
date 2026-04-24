---
trigger: always_on
description: This document provides essential information for AI agents and developers working
---

# AGENTS.md - Perles Development Guide

This document provides essential information for AI agents and developers working
on the Perles codebase. It documents the actual patterns, conventions, and
commands observed in this repository.

## Project Overview

Perles is a terminal-based search and kanban board for [beads](https://github.com/steveyegge/beads)
issue tracking, built in Go using the Bubble Tea TUI framework. It includes a
**multi-agent AI orchestration system** for coordinating AI-powered development workflows.

**Requirements:**

- Go 1.24+
- A beads-enabled project (`.beads/` directory with `beads.db`)
- golangci-lint (for linting)
- mockery (for mock generation)

## Essential Commands

### Build & Run

```bash
make build          # Build the binary with version info
make run            # Build and run
make debug          # Build and run with debug flag (-d)
make install        # Install to $GOPATH/bin
make clean          # Clean build artifacts

./perles            # Run the built binary
./perles -d         # Run with debug mode
./perles -c path    # Use specific config file
./perles -b path    # Specify beads directory
./perles playground # Run the vimtextarea playground
./perles workflows  # List available workflow templates
```

### Testing

```bash
make test           # Run all tests
make test-v         # Run tests with verbose output
make test-update    # Update golden test files for teatest snapshots

# Update specific golden tests
go test -update ./internal/mode/search/...
go test -update ./internal/ui/board/...
go test -update ./internal/ui/shared/vimtextarea/...

# Run specific package tests
go test ./internal/bql/...
go test ./internal/orchestration/v2/...
go test -v ./cmd/...
```

### Code Quality

```bash
make lint           # Run golangci-lint (configured in .golangci.yml)
make mocks          # Generate mocks using mockery
make mocks-clean    # Remove generated mocks
go fmt ./...        # Format code (Go standard)
```

### Version Control

```bash
# The project uses standard Git workflow
# Version is automatically extracted from git tags
git describe --tags --always --dirty  # Version format used
```

## Code Organization

### Directory Structure

```
perles/
├── cmd/                    # CLI commands (cobra)
│   ├── root.go            # Main command setup
│   ├── init.go            # Init command
│   ├── themes.go          # Theme commands
│   ├── workflows.go       # Workflow listing command
│   └── playground.go      # Playground command
├── internal/              # Internal packages (not exported)
│   ├── app/              # Root application model & orchestration
│   ├── application/      # DDD application layer (scoped to registry)
│   │   └── registry/     # Registry service facade (YAML loading, template rendering)
│   ├── domain/           # DDD domain layer (scoped to registry)
│   │   └── registry/     # Pure domain types (Registration, Chain, Node)
│   ├── mode/             # Application modes with Controller interface
│   │   ├── kanban/       # Kanban board mode
│   │   ├── search/       # Search mode
│   │   ├── orchestration/# AI orchestration mode (single-workflow TUI)
│   │   ├── dashboard/    # Dashboard mode (multi-workflow management)
│   │   ├── playground/   # VimTextarea testing playground
│   │   └── shared/       # Cross-mode utilities (Clipboard, Clock)
│   ├── beads/            # Database client and domain models
│   ├── bql/              # Query language implementation
│   │   ├── lexer.go      # Tokenization
│   │   ├── parser.go     # AST building
│   │   ├── executor.go   # Query execution
│   │   └── validator.go  # Query validation
│   ├── cachemanager/     # Generic caching infrastructure
│   ├── git/              # Git executor for worktree operations
│   ├── orchestration/    # AI orchestration system
│   │   ├── client/       # Provider-agnostic headless AI client
│   │   ├── amp/          # Amp CLI integration
│   │   ├── claude/       # Claude Code CLI integration
│   │   ├── codex/        # OpenAI Codex CLI integration
│   │   ├── v2/           # V2 command processor & handlers
│   │   ├── controlplane/ # Multi-workflow control plane
│   │   ├── mcp/          # Model Context Protocol server
│   │   ├── events/       # Process event types
│   │   ├── workflow/     # Workflow template registry
│   │   ├── session/      # Session tracking
│   │   ├── metrics/      # Token usage tracking
│   │   └── message/      # Inter-agent message log
│   ├── pubsub/           # Generic pub/sub event broker
│   ├── ui/               # UI components
│   │   ├── board/        # Kanban board view
│   │   ├── tree/         # Tree view for dependencies
│   │   ├── details/      # Issue details panel
│   │   ├── coleditor/    # Column editor UI
│   │   ├── commandpalette/ # Command palette for orchestration
│   │   ├── modals/       # Modal dialogs
│   │   ├── nobeads/      # Empty state when no .beads DB
│   │   ├── outdated/     # Database version too old state
│   │   ├── shared/       # Reusable UI components
│   │   │   └── vimtextarea/ # Vim-like textarea widget
│   │   └── styles/       # Theme system
│   ├── config/           # Configuration management
│   ├── log/              # Debug logging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zjrosen/perles](https://github.com/zjrosen/perles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
