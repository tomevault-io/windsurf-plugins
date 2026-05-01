---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**multiclaude** is a lightweight orchestrator for running multiple Claude Code agents on GitHub repositories. Each agent runs in its own tmux window with an isolated git worktree, enabling parallel autonomous work on a shared codebase.

### The Brownian Ratchet Philosophy

This project embraces controlled chaos: multiple agents work simultaneously, potentially duplicating effort or creating conflicts. **CI is the ratchet** - if tests pass, the code goes in. Progress is permanent.

**Core Beliefs (hardcoded, not configurable):**
- CI is King: Never weaken CI to make work pass
- Forward Progress > Perfection: Partial working solutions beat perfect incomplete ones
- Chaos is Expected: Redundant work is cheaper than blocked work
- Humans Approve, Agents Execute: Agents create PRs but don't bypass review

## Quick Reference

```bash
# Build & Install
go build ./cmd/multiclaude         # Build binary
go install ./cmd/multiclaude       # Install to $GOPATH/bin

# CI Guard Rails (run before pushing)
make pre-commit                    # Fast checks: build + unit tests + verify docs
make check-all                     # Full CI: all checks that GitHub CI runs
make install-hooks                 # Install git pre-commit hook

# Test (run before pushing)
go test ./...                      # All tests
go test ./internal/daemon          # Single package
go test -v ./test/...              # E2E tests (requires tmux)
go test ./internal/state -run TestSave  # Single test

# Development
go generate ./pkg/config           # Regenerate CLI docs for prompts
MULTICLAUDE_TEST_MODE=1 go test ./test/...  # Skip Claude startup
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLI (cmd/multiclaude)                    │
└────────────────────────────────┬────────────────────────────────┘
                                 │ Unix Socket
┌────────────────────────────────▼────────────────────────────────┐
│                          Daemon (internal/daemon)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ Health   │  │ Message  │  │ Wake/    │  │ Socket   │        │
│  │ Check    │  │ Router   │  │ Nudge    │  │ Server   │        │
│  │ (2min)   │  │ (2min)   │  │ (2min)   │  │          │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└────────────────────────────────┬────────────────────────────────┘
                                 │
    ┌────────────────────────────┼────────────────────────────────┐
    │                            │                                │
┌───▼───┐  ┌───────────┐  ┌─────▼─────┐  ┌──────────┐  ┌────────┐
│super- │  │merge-     │  │workspace  │  │worker-N  │  │review  │
│visor  │  │queue      │  │           │  │          │  │        │
└───────┘  └───────────┘  └───────────┘  └──────────┘  └────────┘
    │           │              │              │             │
    └───────────┴──────────────┴──────────────┴─────────────┘
              tmux session: mc-<repo>  (one window per agent)
```

### Package Responsibilities

| Package | Purpose | Key Types |
|---------|---------|-----------|
| `cmd/multiclaude` | Entry point | `main()` |
| `internal/cli` | All CLI commands | `CLI`, `Command` |
| `internal/daemon` | Background process | `Daemon`, daemon loops |
| `internal/state` | Persistence | `State`, `Agent`, `Repository` |
| `internal/messages` | Inter-agent IPC | `Manager`, `Message` |
| `internal/prompts` | Agent system prompts | Embedded `*.md` files, `GetSlashCommandsPrompt()` |
| `internal/prompts/commands` | Slash command templates | `GenerateCommandsDir()`, embedded `*.md` |
| `internal/hooks` | Claude hooks config | `CopyConfig()` |
| `internal/worktree` | Git worktree ops | `Manager`, `WorktreeInfo` |
| `internal/socket` | Unix socket IPC | `Server`, `Client`, `Request` |
| `internal/errors` | User-friendly errors | `CLIError`, error constructors |
| `internal/names` | Worker name generation | `Generate()` (adjective-animal) |
| `internal/templates` | Agent prompt templates | Template loading and embedding |
| `internal/agents` | Agent management | Agent definition loading |
| `pkg/config` | Path configuration | `Paths`, `NewTestPaths()` |
| `pkg/tmux` | **Public** tmux library | `Client` (multiline support) |
| `pkg/claude` | **Public** Claude runner | `Runner`, `Config` |

### Data Flow

1. **CLI** parses args → sends `Request` via Unix socket
2. **Daemon** handles request → updates `state.json` → manages tmux
3. **Agents** run in tmux windows with embedded prompts and per-agent slash commands (via `CLAUDE_CONFIG_DIR`)
4. **Messages** flow via filesystem JSON files, routed by daemon
5. **Health checks** (every 2 min) attempt self-healing restoration before cleanup of dead agents

## Key Files to Understand

| File | What It Does |
|------|--------------|
| `internal/cli/cli.go` | **Large file** (~5500 lines) with all CLI commands |
| `internal/daemon/daemon.go` | Daemon implementation with all loops |
| `internal/state/state.go` | State struct with mutex-protected operations |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dlorenc/multiclaude](https://github.com/dlorenc/multiclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
