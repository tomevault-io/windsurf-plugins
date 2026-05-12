---
trigger: always_on
description: This file provides guidance to coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working with code in this repository.

## Project Overview

**OAT** (Open Agent Teams) is a lightweight orchestrator for running multiple AI coding agents on GitHub repositories. Each agent runs as its own process with an isolated git worktree, enabling parallel autonomous work on a shared codebase.

### Principles

Multiple agents work simultaneously, potentially duplicating effort or creating conflicts. CI is the quality gate: if tests pass, the code goes in. Progress is permanent.

**Core Beliefs (hardcoded, not configurable):**
- Never weaken CI: Never weaken or disable tests to make work pass; fix the code that causes the failure. Use test-driven verification (targeted tests for changed area; full regression when appropriate).
- Forward Progress > Perfection: Partial working solutions beat perfect incomplete ones
- Redundant work is acceptable: Redundant work is cheaper than blocked work
- Humans Approve, Agents Execute: Agents create PRs but do not bypass review

## Quick Reference

```bash
# Build & Install
go install ./cmd/oat       # Build + install to $GOPATH/bin
# AVOID: go build ./cmd/oat  -- drops ./oat in cwd, shadows $GOPATH/bin/oat

# CI Guard Rails (run before pushing)
make pre-commit                    # Fast checks: build + unit tests + verify docs
make check-all                     # Full CI: all checks that GitHub CI runs
make install-hooks                 # Install git pre-commit hook

# Test (run before pushing)
go test ./...                      # All tests
go test ./internal/daemon          # Single package
go test -v ./test/...              # E2E tests
go test ./internal/state -run TestSave  # Single test

# Development
go generate ./pkg/config           # Regenerate CLI docs for prompts
OAT_TEST_MODE=1 go test ./test/...  # Skip agent startup

# Key environment variables
OAT_FAST_MERGE=false               # Disable daemon auto-merge of green PRs (default: true)
OAT_WORKER_DORMANCY_CAP_MINUTES=30 # Extend worker dormancy cap (default: 15)
OAT_CORE_AGENT_SOFT_TIMEOUT=10     # Minutes before nudging stuck core agents (default: 5)
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLI (cmd/oat)                    │
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
              oat session: <repo>  (one process per agent)
```

### Package Responsibilities

| Package | Purpose | Key Types |
|---------|---------|-----------|
| `cmd/oat` | Entry point | `main()` |
| `internal/cli` | All CLI commands | `CLI`, `Command` |
| `internal/daemon` | Background process | `Daemon`, daemon loops |
| `internal/state` | Persistence | `State`, `Agent`, `Repository` |
| `internal/messages` | Inter-agent IPC | `Manager`, `Message` |
| `internal/prompts` | Agent system prompts | Embedded `*.md` files, `GetSlashCommandsPrompt()` |
| `internal/prompts/commands` | Slash command templates | `GenerateCommandsDir()`, embedded `*.md` |
| `internal/hooks` | agent hooks config | `CopyConfig()` |
| `internal/worktree` | Git worktree ops | `Manager`, `WorktreeInfo` |
| `internal/socket` | Unix socket IPC | `Server`, `Client`, `Request` |
| `internal/errors` | User-friendly errors | `CLIError`, error constructors |
| `internal/names` | Worker name generation | `Generate()` (adjective-animal) |
| `internal/templates` | Agent prompt templates | Template loading and embedding |
| `internal/agents` | Agent management | Agent definition loading |
| `pkg/config` | Path configuration | `Paths`, `NewTestPaths()` |
| `pkg/backend` | **Public** backend abstraction | `ProcessBackend` interface |
| `pkg/agent` | **Public** agent runner | `Runner`, `Config` |

### Data Flow

1. **CLI** parses args → sends `Request` via Unix socket
2. **Daemon** handles request → updates `state.json` → manages agent processes
3. **Agents** run as daemon child processes with embedded prompts and per-agent slash commands (via `~/.oat/agent-config/`)
4. **Messages** flow via filesystem JSON files, routed by daemon

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Root-IO-Labs/open-agent-teams](https://github.com/Root-IO-Labs/open-agent-teams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
