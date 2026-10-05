---
trigger: always_on
description: Operational guidance for coding agents working in this repository. Keep changes small, match the current rewrite architecture, and prefer the documented daemon/API boundaries over behavior from the old TypeScript implementation.
---

# AGENTS.md

Operational guidance for coding agents working in this repository. Keep changes small, match the current rewrite architecture, and prefer the documented daemon/API boundaries over behavior from the old TypeScript implementation.

## Repo layout

- `backend/`: Go rewrite of Agent Orchestrator with the Cobra `ao` CLI, loopback HTTP daemon, services, SQLite storage, lifecycle/reaper, runtime/workspace/agent/tracker adapters, terminal mux, and tests.
- `frontend/`: Electron + React supervisor wired to the daemon through the generated typed client. Treat it as a thin supervisor/UI surface; do not move daemon logic into it.
- `docs/`: Current architecture and status notes. Start here before changing lifecycle, CLI, agents, storage, or daemon behavior.
- `test/`: External smoke/e2e assets, including the CLI fresh-install container check.
- `.github/workflows/`: CI definitions. Mirror these commands locally when possible.

## Commands

From the repo root unless noted:

```bash
npm run lint                         # backend go test ./... + golangci-lint v2.13.2
npm run frontend:typecheck           # frontend TypeScript check
npm run sqlc                         # regenerate backend/internal/storage/sqlite/gen from queries/schema
npm run api                          # regenerate OpenAPI spec + frontend TS types (see API contract changes below)
npx @redwoodjs/agent-ci run --all    # local workflow validation; requires Docker socket
```

Backend-specific checks:

```bash
cd backend
go build ./...
go test ./...
go test -race ./...
go vet ./...
go run ./cmd/ao start
```

Frontend-specific checks:

```bash
cd frontend
npm run typecheck
npm run build
```

When showing or demoing frontend changes, run `ao preview [url]` from inside the session so the change renders in the desktop browser panel (the inspector rail's Browser tab); do not just describe it.

### Desktop lab (running the real Electron app for UI review)

For visual verification that needs the real app rather than `ao preview` or `dev:web`, run the checkout in an isolated worktree against scratch data. Never use the invoking working tree (it may hold unrelated uncommitted work) and never touch the user's real `~/.ao` data.

```bash
git worktree add /tmp/ao-lab <branch>   # the PR branch under review
cd /tmp/ao-lab/frontend
npm ci                                  # never symlink node_modules from another checkout (see below)
npm run build:daemon -- --dev
AO_DATA_DIR=/tmp/ao-lab-data ./node_modules/.bin/electron-forge start
```

- **Run a real `npm ci`.** Symlinking another checkout's `node_modules` makes React resolve from two physical paths and shares the Vite optimizer cache across checkouts: the symptom is a black window with `Invalid hook call` / `useSyncExternalStore` errors in the renderer console. It also writes `.vite/deps` output into the other checkout's tree.
- **Isolate data with `AO_DATA_DIR`.** The lab starts with an empty board; create throwaway sessions inside it to exercise list/detail flows. The Electron `userData` dir already defaults to a dev path, not the production one.
- **Diagnose with `ELECTRON_ENABLE_LOGGING=1`.** Renderer console errors (the cause of most blank windows) land in the launcher's stdout.
- **Killing the forge wrapper is not enough.** The Electron main process cmdline is just `Electron .`, so `pkill -f electron-forge` leaves the app running and the next launch exits silently. Kill the `Electron.app/.../MacOS/Electron` main process explicitly before relaunching.
- A missing-dependency warning for a lazily imported module (for example `mermaid`, which is dynamically imported on first use) does not blank the window; treat it as informational.

## Where to look first

- `README.md`: Current run/config/test quickstart.
- `docs/README.md`: Docs index.
- `docs/documentation-map.md`: Which artifacts are the machine-readable contract layer (`openapi.yaml`, `AGENTS.md`, `skills/`, sqlc `gen/`), what each is source of truth for, and how CI keeps them from drifting.
- `docs/architecture.md`: Backend mental model, package layout, lifecycle/session/service boundaries, and load-bearing rules.
- `docs/STATUS.md`: What is shipped on `main` today and what is still in flight.
- `docs/cli/README.md`: Intended CLI shape, a thin Cobra client over daemon HTTP with no direct storage/runtime access.
- `CLAUDE.md`: Compatibility pointer for Claude Code. It directs agents back to `AGENTS.md`.

For code entry points:

- CLI commands: `backend/internal/cli/*.go`; follow nearby command/test patterns before adding a new style.
- HTTP controllers and DTOs: `backend/internal/httpd/controllers/`.
- Service read/write boundaries: `backend/internal/service/`.
- Domain vocabulary: `backend/internal/domain/`.
- Port contracts: `backend/internal/ports/`.
- SQLite queries/migrations/store: `backend/internal/storage/sqlite/`.
- Generated sqlc code: `backend/internal/storage/sqlite/gen/`.

## Distribution

- The **desktop app** (GitHub Releases) is the canonical, auto-updating install path. Point users there first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OrchestratorInc/agent-orchestrator](https://github.com/OrchestratorInc/agent-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
