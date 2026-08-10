---
trigger: always_on
description: Operational guidance for coding agents working in this repository. Keep changes small, match the current rewrite architecture, and prefer the documented daemon/API boundaries over behavior from the old TypeScript implementation.
---

# AGENTS.md

Operational guidance for coding agents working in this repository. Keep changes small, match the current rewrite architecture, and prefer the documented daemon/API boundaries over behavior from the old TypeScript implementation.

## Repo layout

- `backend/` — Go rewrite of Agent Orchestrator: Cobra `ao` CLI, loopback HTTP daemon, services, SQLite storage, lifecycle/reaper, runtime/workspace/agent/tracker adapters, terminal mux, and tests.
- `frontend/` — Electron + React supervisor wired to the daemon via the generated typed client. Treat it as a thin supervisor/UI surface; do not move daemon logic into it.
- `docs/` — current architecture/status notes. Start here before changing lifecycle, CLI, agents, storage, or daemon behavior.
- `test/` — external smoke/e2e assets, including the CLI fresh-install container check.
- `.github/workflows/` — CI definitions. Mirror these commands locally when possible.

## Commands

From the repo root unless noted:

```bash
npm run lint                         # backend go test ./... + golangci-lint v2.12.2
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

## Where to look first

- `README.md` — current run/config/test quickstart.
- `docs/README.md` — docs index.
- `docs/architecture.md` — backend mental model, package layout, lifecycle/session/service boundaries, and load-bearing rules.
- `docs/STATUS.md` — what is shipped on `main` today and what is still in flight.
- `docs/cli/README.md` — intended CLI shape: thin Cobra client over daemon HTTP, never direct storage/runtime access.
- `CLAUDE.md` — compatibility pointer for Claude Code; it directs agents back to `AGENTS.md`.

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
- **npm still works but is no longer recommended.** `0.10.0` is the final version published to npm; the `@aoagents/ao` package is frozen and will not receive further updates. It remains a legacy on-ramp for users who already have `ao` on their PATH, where `ao start` fetches and opens the desktop build. Do not add features, docs, or flows that treat npm as the intended way to install AO.
- **Exactly one publisher.** Only the designated release conductor runs a real publish, on any channel. Divergent artifacts from multiple publishers made the 28-29 Jul macOS incident unreadable. Use the fork dev loop for test builds. Full rule and rationale: `frontend/docs/desktop-release.md`, "Hard rule: exactly one publisher".
- **Verify macOS artifacts with `frontend/scripts/verify-mac-artifact.sh`, never by hand.** It extracts with `ditto -x -k` and runs `codesign --verify --deep --strict`, `spctl -a -vv -t exec`, `xcrun stapler validate`. Plain `unzip` breaks the seal and yields a convincing false failure; `spctl` without `-vv` prints nothing at all on success.
- **macOS ships both a `.zip` and a `.dmg`.** The dmg is first install only. The zip and `latest-mac.yml` must keep publishing forever: electron-updater cannot install an update from a dmg. macOS differential updates are permanently disabled (full download only); see issues #3151 and #3267.

## Coding conventions

- Keep every change surgical and directly tied to the task. Avoid drive-by cleanup, broad renames, formatting churn, speculative abstractions, and architectural refactors unless the task explicitly asks for them.
- Follow existing Go package boundaries. CLI code should call daemon HTTP routes through shared CLI client helpers; it should not open SQLite, spawn runtimes, or call adapters directly.
- Keep Cobra commands in the relevant command file and table-test them in the style of `backend/internal/cli/*_test.go`.
- Mirror existing response/request DTOs in the CLI instead of importing HTTP controller packages into CLI code, unless the package already establishes that dependency.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Untrivial-ai/agent-orchestrator](https://github.com/Untrivial-ai/agent-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
