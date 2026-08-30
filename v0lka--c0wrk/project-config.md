---
trigger: always_on
description: Guidance for coding agents working on **c0wrk** — a desktop AI coding-agent built with Wails v2 (Go backend + React 19 / Vite 6 / TS frontend).
---

# AGENTS.md

Guidance for coding agents working on **c0wrk** — a desktop AI coding-agent built with Wails v2 (Go backend + React 19 / Vite 6 / TS frontend).

## Security Policy

This project maintains a security policy in [SECURITY.md](./SECURITY.md).
All AI coding agents MUST read and follow SECURITY.md before making changes.
It contains:

- Threat model and trust boundaries
- Secure coding guidelines specific to this project's stack
- Hard constraints and forbidden patterns for AI agents
- Vulnerability reporting procedures
- Agentic security controls (OWASP Top 10 for Agentic Applications ASI01–ASI10)

Any code contribution that violates the rules in SECURITY.md will be rejected.

## Specifications

Detailed system specs live in `specs/`. Before making structural changes, read the relevant spec:

- Start with `specs/INDEX.md` to find the right document for your task.
- `specs/META.md` defines spec formats and update rules — read before creating/updating specs.
- `specs/contracts/` define cross-boundary interface rules.
- `specs/domains/` explain subsystem behavior and invariants.
- `specs/decisions/` explain why things are designed the way they are.

## Project shape

- Go module: `github.com/v0lka/c0wrk` (root: `core/`, `backend/`, `desktop/`, `frontend/`). Binary/app name is `c0wrk-desktop` (see `wails.json`).
- Entry point: `main.go` → `desktop.NewApp()` → Wails runs with `OnStartup = app.Startup` (`desktop/startup.go`). Build metadata is injected into `core/version` by Makefile/release `-ldflags` (`Version`, `GitCommit`, `BuildDate`).
- Go `1.26.3` (single root module; `go.mod` at repo root). Frontend uses React 19, Tailwind v4, Vite 6, TS ~5.7.

### Layered architecture (import direction matters)

```
desktop/   Wails bindings + app lifecycle (UI callbacks)  →  depends on backend, core
backend/   "Application" ViewModel, config, session mgr, persistence, MCP installer, workspace watcher
core/      Orchestrator / planner / router / reflector / tool registry / MCP gateway / vector index / proxy / c0wrk-specific tools
frontend/  React UI; talks to Go via `frontend/wailsjs/go/desktop/App` (generated)
```

Rule enforced by layout: `backend/` and `desktop/` import `core` directly. `core/` remains the primary consumer of sp4rk. No convenience re-export layers exist — all types are imported from their source packages. See ADR-008.

## Commands

Use the Makefile; it handles platform-specific ONNX Runtime bootstrap across the single root Go module and the frontend:

- `make test` — `go test ./...` (root) + `cd frontend && npm test` (vitest)
- `make lint` — `make fmt-check` + `golangci-lint run` (root) + `cd frontend && npm run lint` (config at `.golangci.yml`, v2 schema)
- `make fmt-check` — fails when `gofmt -l` reports any Go file under the root package, `internal/`, `core/`, `backend/`, or `desktop/`
- `make build` — installs frontend deps, runs `wails build` with version ldflags, then `make fetch-onnx` + `make fetch-embedding-model`
- `make dev-desktop` — Vite dev server only (`cd frontend && npm run dev`); for full hot-reload use `wails dev` from repo root
- `make fetch-onnx` — downloads ONNX Runtime 1.24.1 into `.cache/` and copies it into the platform build output. **Required after every direct `wails build`** or the app won't launch.
- `make bump` — resolves the latest `github.com/v0lka/sp4rk` remote commit and updates the module with `GOWORK=off`; use only at the release point of a cross-repo development cycle
- `make clean` — removes `build/bin`, `.cache`, `frontend/dist`

Frontend-only: `cd frontend && npm run lint | build | dev | test`. Frontend tests use **vitest** (`npm test` / `npm run test:watch`); test files live alongside source (`*.test.ts`).

### Focused Go workflows

- Single package (root module): `go test ./core/...`
- Single test (root module): `go test ./core -run TestOrchestrator_PlanExecuteMode -v`
- Tests use in-package style (`package agent`, not `agent_test`); many packages have a `testhelpers_test.go`.

## Config & runtime

- Runtime config lives at `~/.c0wrk/config.yaml` (default dir constant `config.DefaultAgentDir = ".c0wrk"`). `config.example.yaml` is the authoritative reference for every tunable (LLM providers, executor loop caps, compaction thresholds, tool limits, timeouts, security policies, Small-LLM profile, vector index, experimental features, updates).
- **Experimental gate** (`experimental.enabled`, default false) is all-or-nothing. It gates RESEARCH and the entire Small-LLM profile; disabling it leaves stored per-feature config/artifacts intact but makes both features ineffective and hides their UI. `backend/frontend_api_research.go` owns RESEARCH activation, workspace containment, non-destructive skill seeding, watcher setup, and disable semantics.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [v0lka/c0wrk](https://github.com/v0lka/c0wrk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
