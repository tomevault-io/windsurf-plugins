---
trigger: always_on
description: Decision coordination layer for multi-agent AI systems ("version control for AI decisions").
---

# ashita-ai/akashi

Decision coordination layer for multi-agent AI systems ("version control for AI decisions").

## Tech stack

- **Server:** Go 1.26, stdlib `net/http` (Go 1.22+ routing), no framework
- **Database:** PostgreSQL 18 + pgvector + TimescaleDB, Atlas for migrations
- **Auth:** Ed25519 JWT + Argon2id API keys, RBAC (platform_admin > org_owner > admin > agent > reader)
- **UI:** React 19, TypeScript, Vite, Tailwind CSS (embedded via `go:embed` with `ui` build tag)
- **SDKs:** Go, Python, TypeScript (in `sdk/`)
- **Testing:** `testing` + testify assertions, testcontainers-go for integration tests
- **Lint:** golangci-lint v2.11.0, Atlas migrate validate

## First-time setup

```sh
make install-hooks   # installs Claude Code hooks (akashi-trace reminder after git commit)
```

This registers a `PostToolUse` hook that fires after every `git commit` and reminds you to call `akashi_trace`. Run once per machine; safe to re-run.

## Commands

**Before every commit (mandatory, CI rejects failures):**
```sh
make preflight
```

This is `ci.yml`'s build job minus the tests: tidy + go.mod diff, doc/config consistency,
Atlas migration validation, `go build`, the **lite build** (`-tags lite ./cmd/akashi-local`),
lint, and vet. No Docker, no database, no API keys. The Makefile target is the only definition —
do not restate the command list here, which is how the two CI-enforced gates went missing from
it for months. The raw commands are kept as a comment above the target.

**Before every push (mandatory, CI runs with `-race`):**
```sh
go test -race -count=1 ./...                       # unit tests only (fast, no containers)
go test -race -count=1 -tags integration ./...     # full suite (unit + integration, requires Docker)
```

**Build:**
```sh
go build ./...                           # without UI
cd ui && npm ci && npm run build && cd .. # build UI assets first
go build -tags ui ./...                  # with embedded React SPA
make ci                                  # full local CI mirror
```

**If go mod tidy changes go.mod/go.sum**, stage them in the commit.
**If atlas validate fails**, run `atlas migrate hash --dir file://migrations` and stage `migrations/atlas.sum`.
**golangci-lint location:** `~/go/bin/golangci-lint` (install: `go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.11.0`).

## Project structure

```
cmd/akashi/          Server entrypoint. Config loading, dependency wiring, signal handling.
cmd/akashi-local/    Local-lite MCP server (SQLite, stdio transport, zero-infra). See ADR-009.
cmd/eval-conflicts/  Evaluation harness for conflict detection precision/recall. Only
                     --mode=benchmark runs standalone; --mode=validator and --mode=scorer
                     need a running server, and --mode=gold needs AKASHI_DB_DSN plus a
                     populated conflict_gold_labels table. See docs/conflict-detection.md.
internal/
  server/            HTTP handlers (handlers*.go), middleware (middleware.go), SSE broker.
  storage/           PostgreSQL queries. One file per entity (decisions.go, agents.go, events.go...).
  storage/sqlite/    SQLite storage backend for local/lite mode. Carries NO build tag — it
                     compiles in every build and is simply not imported outside cmd/akashi-local.
  service/           Business logic. decisions/ (trace pipeline), embedding/, quality/, query/, search/,
                     trace/ (event buffer, WAL), autoassess/, autoresolve/, tracehealth/.
  model/             Domain types. Decision, AgentEvent, Alternative, Evidence, etc.
  config/            Env var loading and validation.
  auth/              JWT issuing/verification, API key hashing (Argon2id).
  authz/             RBAC enforcement, grant cache, access filtering.
  compact/           Compact representation utilities.
  conflicts/         Embedding-based conflict detection + LLM validation.
  ctxutil/           Context utility helpers including audit context.
  integrity/         SHA-256 content hashes, Merkle tree proofs.
  mcp/               MCP server: tool definitions, resources, prompts, session tracking.
  search/            Qdrant vector search with PostgreSQL text fallback.
  ratelimit/         Pluggable token bucket rate limiter.
  telemetry/         OpenTelemetry setup (traces + metrics).
  testutil/          Shared test helpers (testcontainers, test DB, test logger).
migrations/          SQL files (001, 022..111). Atlas-managed checksums.
adrs/                Technical architecture decision records (ADR-001 through ADR-018).
sdk/                 Go, Python, TypeScript client SDKs.
ui/                  React 19 SPA (audit dashboard). Embedded via go:embed when built with -tags ui.
docs/                Configuration reference, runbook, conflict-detection operator guide,
                     quality scoring, GDPR erasure, diagrams. README's Docs table is the index.
```

## Architecture patterns

**Multi-tenancy via org_id.** Every query MUST include `AND org_id = $N`. There are 400+ org_id references across the storage layer. Missing one is a data leak. When adding a new query, always scope by org_id.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ashita-ai/akashi](https://github.com/ashita-ai/akashi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
