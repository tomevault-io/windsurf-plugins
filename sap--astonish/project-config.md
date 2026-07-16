---
trigger: always_on
description: This document is the **root** guidance file for agentic coding systems working on the Astonish codebase. Deeper AGENTS.md files exist under significant subsystems (see [Hierarchical AGENTS.md Index](#hierarchical-agentsmd-index)) — always consult the deepest one whose scope covers the files you are editing.
---

# AGENTS.md - Agent Coding Guidelines

This document is the **root** guidance file for agentic coding systems working on the Astonish codebase. Deeper AGENTS.md files exist under significant subsystems (see [Hierarchical AGENTS.md Index](#hierarchical-agentsmd-index)) — always consult the deepest one whose scope covers the files you are editing.

## Project Overview

Astonish is a **multi-tenant AI agent platform** written in Go with a React/TypeScript UI. It uses Google's Agent Development Kit (ADK) and provides three top-level modes:

- **Personal / CLI**: `astonish chat` — single-user, SQLite, in-process agent runtime.
- **Studio (HTTP + SPA)**: `astonish daemon run` → serves REST/SSE API under `/api/*` and the embedded React SPA. This is what people usually call "Astonish Studio".
- **Platform**: multi-tenant PostgreSQL, envelope encryption, org/team/personal scoping, sandboxed execution over Kubernetes/OpenShell/Incus, channel adapters (Slack/Telegram/Email), scheduler, remote CLI.

**Stack:**
- **Backend**: Go 1.26 (see `go.mod`; toolchain go1.26.x), ADK (`github.com/google/adk-go`), Ent ORM (`entgo.io/ent`), bubbletea (TUI), pgx (Postgres), `atlas` for migrations, `mcp-go-sdk`, starlark, Kubernetes/Incus client libraries.
- **Frontend**: React 19.2 with **mixed TypeScript (`.ts`/`.tsx`) and JSX (`.jsx`)** (the app entry is `web/src/main.tsx`), Vite 7.2, Tailwind CSS 4.1, Vitest. `npm run build` runs `tsc --noEmit` before Vite.
- **Build System**: Make (`Makefile`, `Makefile.integration`). Pre-commit hook (`.githooks/pre-commit`) enforces Atlas migration integrity.
- **Sandbox / Isolation**: Incus (default, LXC), Kubernetes, OpenShell (gRPC gateway with Landlock/seccomp), plus an in-memory mock for tests.

## Build / Lint / Test Commands

### Go Backend
```bash
# Build everything (UI + Go binary)
make build-all

# Build Go binary only
make build

# Build React UI only
make build-ui

# Run Go application
go run .

# Run Astonish Studio (HTTP + SPA on :9393 by default)
make studio              # Production mode (serves embedded UI)
make studio-dev          # Dev mode (live UI reload on http://localhost:5173)

# Tests (tiered)
make test-unit           # Go + frontend unit tests (fast, no external deps)
make test-integration    # Integration tests (needs ASTONISH_TEST_DSN)
make test-e2e            # Full E2E (needs ASTONISH_TEST_DSN + provider API key + kubectl + helm)
make test-e2e-sqlite     # E2E in SQLite mode (no ASTONISH_TEST_DSN, still needs provider key + k8s sandbox)
make test-e2e-openshell  # E2E against the OpenShell sandbox backend
make test-e2e-inspect    # Leave a long-lived inspector server on :9394 after the run
make test-e2e-inspect-stop # Stop the inspector

# Run single test
go test ./pkg/tools -run TestFileTree
go test -v ./pkg/tools -run TestFileTree  # Verbose

# Lint
golangci-lint run        # Full lint check (bug-finders only; see .golangci.yml)
# Prefer: make lint      # config verify + run; version must match .golangci-version (CI)

# Atlas migrations (schema/*.sql, pkg/store/*/migrations)
make migrate-diff        # Generate a new migration by diffing ent schemas
```

Full e2e infra (docker-compose, k8s namespace bring-up, PVCs, inspector state) is documented in `tests/AGENTS.md`.

### React Frontend (in `web/` directory)
```bash
cd web

# Development server with hot reload
npm run dev              # http://localhost:5173

# Build for production (runs tsc --noEmit first)
npm run build

# Lint
npm run lint

# Unit tests (Vitest)
npm test
```

### Quick Reference
- Single test: `go test ./pkg/path -run TestFunctionName`
- Verbose test: `go test -v ./pkg/path -run TestFunctionName`
- Run specific package: `go test ./pkg/path`
- Run with race detector: `go test -race ./pkg/path`
- Rebuild the e2e-inspector when backend code changed: `make test-e2e-inspect` handles this automatically.

## Go Code Style

- **Imports**: stdlib → external → internal, with blank lines between groups.
- **Naming**: `PascalCase` for exports, `camelCase` for private, lowercase packages.
- **Tags**: `yaml` and `json` with `omitempty` for optional fields.
- **Errors**: return as last value, check immediately, wrap with `fmt.Errorf` when needed. Never suppress with `_`.
- **Interfaces**: minimal, defined **near use** (e.g., `RunnableTool`, `ToolWithDeclaration` in `pkg/tools`, `Backend` in `pkg/sandbox`, `Channel` in `pkg/channels`).
- **Testing**: `*_test.go` same package, table-driven tests, `os.MkdirTemp` with cleanup. Integration/e2e tests use build tags (`//go:build integration`, `//go:build e2e`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP/astonish](https://github.com/SAP/astonish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
