---
trigger: always_on
description: Use `docs/design/overall.md` as the high-level architecture map when you need system context. Do not update internal design docs for routine code changes. Update `docs/design` only when a change materially affects architecture, durable product contracts, data models, APIs, operational procedures, or when the user explicitly asks for a design update.
---

## Documentation

Use `docs/design/overall.md` as the high-level architecture map when you need system context. Do not update internal design docs for routine code changes. Update `docs/design` only when a change materially affects architecture, durable product contracts, data models, APIs, operational procedures, or when the user explicitly asks for a design update.

Update public Fumadocs (`docs/public`) only for significant user-facing behavior, setup, API, or integration changes. Most PRs should not need documentation updates.

## Debugging Production

When investigating bugs or unexpected behavior, three Make targets give read-only access to prod. All require `SSH_KEY` (defaults to `~/.ssh/143-deploy`) and resolve hosts/credentials via sops from `.env.production.enc` in the private secrets checkout (`SECRETS_DIR`, default `../143-infra` — see docs/secrets/README.md).

### Querying the database

- **`make db-query Q='SELECT ...'`** — runs a one-shot SQL query against the prod Postgres as the `readonly` role. SELECT-only, every connection is a read-only txn, bounded by a 30s `statement_timeout`. Use single quotes around `Q` and escape literal `$` as `$$` (Make eats single `$`). For an interactive session, use `make db-psql`.

### Searching logs

Logs are shipped via Vector to a VictoriaLogs instance and visualized in Grafana. Use the CLI for scripted/agent searches and the UI for interactive exploration:

- **`make logs-query Q='<LogsQL>' [LIMIT=100]`** — runs a one-shot LogsQL query against VictoriaLogs and prints NDJSON to stdout. Always include a `_time:` filter — without one VictoriaLogs scans the full 30-day retention. Common fields: `service`, `level`, `org_id`, `agent_run_id`, `request_id`, `trace_id`. Examples:

  ```bash
  make logs-query Q='service:api AND level:error AND _time:[now-1h,now]'
  make logs-query Q='agent_run_id:"run-abc123" AND _time:[now-24h,now]' LIMIT=500
  make logs-query Q='"timeout waiting for sandbox" AND _time:[now-15m,now]' | jq -r '.message'
  ```

  See `docs/design/implemented/47-logging-victorialogs.md` for more examples and the [LogsQL reference](https://docs.victoriametrics.com/victorialogs/logsql/).

- **`make logs`** — opens an SSH tunnel to the prod Grafana instance at <http://localhost:9999> for interactive UI-based exploration. Press Ctrl+C to close the tunnel. Prefer `make logs-query` for anything scriptable.

## Backend Architecture (Go)

**Key libraries**: `go-chi/chi` (router), `jackc/pgx` (Postgres driver + connection pooling), `rs/zerolog` (structured logging), `go-playground/validator` (request validation), `golang-migrate/migrate` (schema migrations). See `docs/design/implemented/02-api-server.md` for full dependency list.

**Direct pgx store functions for all DB queries**: One store struct per domain area in `internal/db/` (e.g. `issues.go`, `agent_runs.go`, `jobs.go`). All stores accept a `DBTX` interface (satisfied by both `pgxpool.Pool` and `pgx.Tx`). Use `pgx.CollectRows` + `pgx.RowToStructByName` for list queries. SQL lives as string literals inside Go functions, co-located with scanning and error handling. No ORM, no codegen, no separate `.sql` files.

**Service layer**: Handlers call services, services call the DB layer. Business logic belongs in `internal/services/`, never in HTTP handlers. Services are defined as interfaces for testability (mock with `go.uber.org/mock`).

**LLM prompts live in Go templates**: All LLM system prompts must be stored as `.template` files in `internal/prompts/templates/`, rendered via the `prompts` package (`internal/prompts/prompts.go`). Never inline prompt strings as Go constants or string literals in service code. Add a corresponding exported render function in `prompts.go` for each new template.

**Logging**: Use `zerolog` for all log output. Never use `fmt.Printf` or `log.Println`. Logs are JSON-structured and shipped to Mezmo.

**Error handling**: Never discard errors with `_ =` in Go or empty `.catch()` in TypeScript. If an error cannot be propagated (e.g., best-effort cleanup after the main operation succeeded), log it at `Warn` level with context. In HTTP handlers, use `zerolog.Ctx(r.Context())` to get the request-scoped logger (enriched with org_id, user_id, request_id by the `LogContext` middleware). In services, use `s.logger`. If an error CAN be propagated, return it — prefer bubbling errors to the top of the call stack. Transaction rollback in `defer` is the one exception: `defer func() { _ = tx.Rollback(ctx) }()` is acceptable because rollback after commit is a no-op. Frontend: at minimum log with `console.error`; prefer surfacing errors through TanStack Query error states.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [assembledhq/143](https://github.com/assembledhq/143) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
