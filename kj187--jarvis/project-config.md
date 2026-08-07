---
trigger: always_on
description: You are a developer working on Jarvis, a web frontend for Prometheus
---

# AGENTS.md — Jarvis

You are a developer working on Jarvis, a web frontend for Prometheus
Alertmanager. This file is the **single entry point for every AI agent**
(Claude Code, GitHub Copilot, Codex, …) and contains the minimum context
needed for any task. Deep, task-specific references live in `.agents/` —
load them on demand via the [Task Router](#task-router--load-on-demand)
below. Never duplicate content from those files here or elsewhere; reference
it instead.

## What Jarvis Is

Jarvis polls all configured Alertmanager clusters, stores every alert
lifecycle in SQLite or PostgreSQL, keeps the current poll snapshot in an
in-memory store, and pushes updates via WebSocket to the frontend. Users can
view, filter, silence, claim, and comment on alerts.

| Layer | Stack |
|---|---|
| Backend | Go 1.25+ · Echo v4 · module `github.com/kj187/jarvis/backend` |
| Frontend | React 19 · TypeScript (`strict`) · Vite 8 · Zustand v5 · TanStack Query v5 · Tailwind v4 |
| Database | SQLite (`modernc.org/sqlite`) or PostgreSQL (`pgx/v5`) — selected by `JARVIS_DB_DSN` prefix, both pure Go (no CGO) |
| Image | Single container: frontend embedded into the Go binary at build time (`//go:build prod` + `embed.FS`), distroless base |

Repository layout:

- `backend/` — Go backend (`internal/api`, `internal/history`, `internal/alertmanager`, `internal/auth`, `internal/ws`, …)
- `frontend/` — React app (`src/components`, `src/hooks`, `src/lib`, `src/store`, `e2e/`)
- `charts/jarvis/` — Helm chart (+ helm-unittest tests under `tests/`)
- `docs/` — user-facing documentation (not AI context, except `docs/testing-e2e.md` and `docs/scope.md`)
- `scripts/` — E2E runner, mock-OIDC config, manual test-alert/silence fixtures
- `.agents/` — task-specific AI reference files (routed below)
- `Makefile` — canonical entry for dev stack, tests, security scans, fixtures (`make help`)

## Task Router — load on demand

Load the referenced file **before** starting the matching task. Do not guess
details that these files own.

| Task | Load |
|---|---|
| Data model, DB schema, API endpoints, component tree, stores, WS events, auth, config env vars, alert state machine, technology decisions | `.agents/architecture.md` |
| Adding a feature: new endpoint, new component, new WS event, new cluster parameter (TDD checklist) | `.agents/add-feature.md` |
| Judging whether a feature idea fits the project scope (scope gate) | `docs/scope.md` |
| Triaging a GitHub feature-request issue against the scope, drafting a reply | `.agents/scope-triage.md` |
| Writing or running tests, test matrix, test utilities, CI pipeline | `.agents/testing.md` |
| E2E / screenshot stack: Playwright specs, fixtures, auth modes, `compose.e2e.yml` | `docs/testing-e2e.md` |
| Database backends, multi-replica HA (leader election, snapshot distribution, WS fanout, failover), Kubernetes deployment, SQLite → PostgreSQL migration | `docs/persistence.md` |
| Cutting a release — **only when the user explicitly asks** | `.agents/release.md` |
| Security audit, new-code security checklist, security tooling | `.agents/security.md` |
| Debugging surprising behavior — check before re-deriving a known gotcha | `.agents/lessons.md` |

Tool-specific entry points map to the same files (no duplicated content):

- **Claude Code**: `CLAUDE.md` includes this file; `/project:architecture`,
  `/project:add-feature`, `/project:testing`, `/project:release`,
  `/project:security-check`, `/project:scope-triage` include the
  corresponding `.agents/` file.
- **GitHub Copilot**: `.github/copilot-instructions.md` is a symlink to this file.
- **Codex**: reads `AGENTS.md` natively.

## Critical Invariants — NEVER break

1. **Grace Period (`max(60s, 2×JARVIS_POLL_INTERVAL)`)**: Alert seen again
   within the grace period after `resolved` → reopen old event, create **no**
   new one. Prevents ghost-resolve entries on poll misses. Configured on
   `Store` via `SetGracePeriod` (`cmd/jarvis/main.go`, derived from
   `cfg.PollInterval`) so a poll interval ≥ 60s can't make a single missed
   poll permanently split one episode into two — a fixed 60s window could
   never absorb a miss at intervals that long. `Recorder.claimReleaseDelay`
   must in turn exceed the grace period (also derived in `main.go`), or the
   delayed claim-release check could run before a grace-period-eligible
   re-fire has had a chance to reopen the event.
2. **Increment `occurrence_count` only on second firing**: Not on the very
   first occurrence — only when `hadPreviousEvent = true`.
3. **`getEffectiveAlertState`**: Alert `suppressed` + **all** active silences
   covering it (`status.silencedBy` can hold more than one) have ≤15 min
   until expiry → returns `active`. Must consider every covering silence, not
   just the first one found in `silencedBy` — a longer-running second silence
   still keeps the alert suppressed. This logic **only** in
   `lib/alertUtils.ts` — never duplicate.
4. **Filter functions exclusively in `lib/alertUtils.ts`**:
   `getFilterableLabels`, `matchesLabelMatchers`, `safeRegex` — no copy-paste
   into components.
5. **Route order in Echo router**: `/api/v1/alerts/groups` must be registered
   **before** `/api/v1/alerts/:fingerprint/*`, otherwise `groups` is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kj187/jarvis](https://github.com/kj187/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
