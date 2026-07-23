---
trigger: always_on
description: <!-- bd-doctor-divergence: ok -->
---

# CLAUDE.md

<!-- bd-doctor-divergence: ok -->
<!-- AGENTS.md and CLAUDE.md intentionally serve different audiences:
     AGENTS.md is generic agent/bd onboarding (any harness); CLAUDE.md
     is Claude-specific project architecture, build commands, and
     doc-quality gates. The divergence is by design — bd doctor's
     "Agent Doc Divergence" warning is opted out here. -->

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gastown Viewer Intent** is a local-first Mission Control dashboard for **Beads** (a local issue tracker with dependency support) and **Gas Town** (a multi-agent orchestrator). It provides board views, dependency graphs, agent status dashboards, molecule tracking, and convoy progress via an HTTP daemon, TUI, and React Web UI.

## Build & Development Commands

```bash

make dev              # Daemon (localhost:7070) + web (localhost:5173) in parallel
make daemon           # Daemon only
make web              # Web dev server only (Vite hot reload)
make tui              # TUI client (requires running daemon)
make build            # Build web, copy to internal/api/web_dist, then build Go binaries
make test             # Go tests + web lint
make clean            # Remove bin/, dist/, web/dist/, internal/api/web_dist/

# Go tests

go test -v ./...                         # All tests
go test -v ./internal/beads/...          # Single package
go test -v -run TestParseIssueList ./internal/beads/...  # Single test

# Web

cd web && npm run dev       # Dev server
cd web && npm run build     # TypeScript check + Vite build
cd web && npm run lint      # ESLint

# Verify daemon

curl http://localhost:7070/api/v1/health
```

## Architecture

Two adapters feed data into a single HTTP server with a security
middleware chain in front:

- **Beads Adapter** (`internal/beads/`): Shells out to `bd` CLI for issue data. Never parses `.beads/` files directly. Uses the `Executor` interface (`DefaultExecutor` for production, `MockExecutor` for tests). Surfaces `bd memories`, `bd dolt status`, `bd human list` in addition to the standard issue/board/graph routes.
- **Gastown Adapter** (`internal/gastown/`): Reads Gas Town filesystem at `~/gt` and shells to `gt` CLI. Molecules now read from `gt wisps list --json` (gt 0.9 surface); legacy `.beads/molecule.json` file reads were removed in `gastown-7fq`.
- **Security middleware** (`internal/api/security.go`): Origin allowlist (DNS-rebind + CSRF defense), session token at `~/.config/gvid/token` (mode 0600), loopback bind enforcement at `Start()`. See `THREAT_MODEL.md` for the full model.
- **Memory redaction** (`internal/api/memoryredact.go`): Applies the partner-name + secret-pattern denylists from `000-docs/005-PP-POLICY-memories-classification-2026-05-24.md` before any memory crosses the HTTP boundary.

Both adapters are interface-based for testability. The `Server` (`internal/api/server.go`) composes both and registers routes on `net/http.ServeMux` using Go 1.22+ method routing (`"GET /api/v1/issues/{id}"`).

**Data flow**: Web UI/TUI → Origin allowlist → CORS → HTTP API (gvid :7070) → Adapters → `bd`/`gt` CLI + filesystem.

**SSE**: The `SSEBroker` (`internal/api/sse.go`) manages client connections with heartbeat at `/api/v1/events`.

## Key Design Decisions

- **Fail-fast**: If `bd` not found, return 503 `BD_NOT_FOUND`. If `.beads/` not initialized, return 503 `BEADS_NOT_INIT`. Every beads handler calls `checkBeadsInitialized()` first.
- **CLI shelling, not file parsing**: Both adapters shell to their respective CLIs rather than parsing internal state files. This keeps the viewer decoupled from internal formats.
- **No external router**: Uses stdlib `net/http.ServeMux` with Go 1.22+ pattern matching. No Gin/Chi/Echo.
- **CORS + Origin allowlist**: CORS headers configured for `http://localhost:5173`; a hard Origin allowlist middleware runs outermost and rejects mismatched cross-origin requests with 403 `ORIGIN_REJECTED`. Native clients (no `Origin` header) bypass the gate by design.
- **Memories panel is read-only-forever**: Council Q2 architectural invariant. Zero state-mutating endpoints under `/api/v1/memories/*`. The bd CLI is the canonical writer. A test (`TestMemoriesHandler_NoPOSTRouteRegistered`) tripwires this rule.
- **Sync pill never errors**: `/api/v1/sync` always returns 200 with a `DoltSyncState` body; failure cases are encoded as `health: "unknown"` with a tooltip string. The header pill must never break the dashboard.

## Testing

Prefer integration tests that hit the real `bd` CLI over mocks. Parser tests (`parser_test.go`) test pure functions and need no CLI. Adapter tests should use `DefaultExecutor` against real beads state when possible. `MockExecutor` exists but is a last resort, not the default approach.

## API Routes

Two route groups defined in `server.go:registerRoutes()`:

- **Beads** (`/api/v1/`): health, issues, board, graph, events, sync, human, memories (+ search + by-key)
- **Gas Town** (`/api/v1/town/`): status, rigs, agents, convoys, molecules, mail

Graph endpoint supports `?format=json` (default) and `?format=dot` (Graphviz DOT). Memory endpoints accept `?reveal=true` to opt into un-redacted content per `005-PP-POLICY`.

## Web UI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremylongshore/gastown-viewer-intent](https://github.com/jeremylongshore/gastown-viewer-intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
