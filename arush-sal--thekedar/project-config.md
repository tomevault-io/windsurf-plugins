---
trigger: always_on
description: This file provides guidance to Codex, Claude Code, and other AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to Codex, Claude Code, and other AI coding agents working in this repository.

---

## START HERE — Onboarding

All authoritative engineering reasoning memory lives in:

ob_vault/

This vault is the architecture intent layer for the system.

Treat:

repo = implementation truth
ob_vault = reasoning truth

Agents must consult the vault before modifying lifecycle behavior, architecture, orchestration logic, or execution semantics.

Start with:

ob_vault/00 - Home.md
ob_vault/Handoff/Agent Handoff.md
ob_vault/Project/State.md
ob_vault/Architecture/core/
ob_vault/Phases/Phase N - *.md

---

## Project

**thekedar** is a Go-based agent orchestrator (AGPL-3.0) that manages AI coding agents (Claude Code, Codex) through a central manager. The manager analyzes tasks using an LLM, spawns agents (dev/qa/review), and coordinates them via native agent-teams and terminal polling. v1 runs locally via tmux. Single project per instance.

**Core Value:** Autonomous dev loop coordination — from task intake through agent spawning, feedback routing, and PR merge — without human intervention beyond initial task assignment.

## Build & Test

```bash
go build ./...                                 # Build all packages
go test ./...                                  # Run all tests
go test -race -count=1 ./...                   # Full suite with race detection
go test ./internal/storage -run TestName -v    # Single test
go vet ./...                                   # Static analysis
golangci-lint run ./...                        # Lint (v2 config in .golangci.yml)
```

---

## Constraints

- **Language:** Go, idiomatic, standard layout (`cmd/`, `internal/`)
- **License:** AGPL-3.0
- **v1 runtime:** tmux only (no K8s/Docker)
- **v1 agents:** Claude Code + Codex CLI only
- **v1 scope:** Single project per instance
- **Testing:** TDD mandatory — write failing test first, then implement. Table-driven tests with testify.
- **API contract:** JSON envelope `{"data": ...}` or `{"error": {"code": N, "message": "..."}}` (see `internal/api/response.go`)
- **Handlers must be thin.** Business logic lives in domain packages (runtime, state, storage), not in HTTP handlers.

---

## Package Map

```
cmd/thekedar/              CLI entry point
internal/api/              REST API (chi v5 router)
internal/config/           Configuration (Viper)
internal/events/           EventBus pub/sub
internal/models/           Domain models (Agent, Task, Event)
internal/poller/           Terminal monitoring + pattern detection
internal/runtime/          Agent lifecycle (tmux, discovery, worktrees)
internal/state/            Agent FSM (qmuntal/stateless)
internal/storage/          Store interface + SQLiteStore
sqlc/                      SQL schema, queries, generated code
.github/workflows/ci.yml   CI pipeline
```

---

### API Routes

```
GET/POST       /api/v1/agents
GET/PATCH/DELETE /api/v1/agents/{id}
POST           /api/v1/agents/{id}/state
POST           /api/v1/agents/{id}/spawn
POST           /api/v1/agents/{id}/kill
GET            /api/v1/agents/{id}/terminal  (WebSocket)
GET/POST       /api/v1/tasks
GET/PATCH      /api/v1/tasks/{id}
GET            /api/v1/events                (SSE)
```

---

### State Machine

```
States: Idle, Working, AwaitingInput, Blocked, Review , Exited

Idle → Working ⇄ AwaitingInput ⇄ Blocked ⇄ Review → Exited
```

---

## Tech Stack

| Component | Library | Version |
|-----------|---------|---------|
| HTTP router | go-chi/chi | v5.2.5 |
| WebSocket | coder/websocket | v1.8.x |
| SSE | r3labs/sse | v2.10.x |
| CLI | spf13/cobra | v1.8.x |
| Config | spf13/viper | v1.19.x |
| Database | modernc.org/sqlite | v1.37+ |
| SQL codegen | sqlc | v1.30+ |
| Migrations | golang-migrate | v4 |
| FSM | qmuntal/stateless | v1.7.x |
| tmux | GianlucaP106/gotmux | v0.5.0 |
| ANSI strip | charmbracelet/x/ansi | v0.11.x |
| Testing | stretchr/testify | v1.11.x |
| Logging | log/slog | stdlib |
| Frontend (Phase 8) | React 19 + Vite 6 + shadcn/ui + xterm.js + Zustand | — |

---

## Known Residual Issues

1. **Spawn route mismatch:** `POST /api/v1/agents/{id}/spawn` ignores the URL `{id}`. Fix: change route to `POST /api/v1/agents/spawn`.
2. **Poller doesn't emit `terminal_update`:** WebSocket transport works but Poller doesn't publish `terminal_update` events to EventBus. Fix: add `EventBus.Publish` in Poller when content changes.

---

## Working Conventions

- Use idiomatic Go — explicit, no magic
- Keep handlers thin — business logic in domain packages
- Preserve the shared response envelope (`internal/api/response.go`)
- TDD: failing test first, then implement
- Do not silently change API contracts without updating tests
- Do not revert unrelated work
- Keep branch state green after each commit
- Update `ob_vault/Project/State.md` when phases complete
- Update the Obsidian vault continuously as work progresses. Conversations, decisions, context, research, findings, implementation notes, caveats, gaps, and phase status changes should be captured in `ob_vault/` so future Codex/Claude sessions can recover project state without re-discovery.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arush-sal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
