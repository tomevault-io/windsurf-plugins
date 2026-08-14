---
trigger: always_on
description: Guidance for AI coding agents working in this repo. Humans, see CONTRIBUTING.md.
---

# AGENTS.md

Guidance for AI coding agents working in this repo. Humans, see CONTRIBUTING.md.

Needs Go 1.25+, Postgres 14+, Node 22+, Docker (integration tests only).

## Non-negotiables

<feedback_loop>
**Tests and the local feedback loop must be green before any push.** That means, at minimum:

- `go build ./...` succeeds
- `golangci-lint run` succeeds (matches the CI lint job)
- `go test -race ./...` (unit) succeeds
- `go test -tags=integration -race ./...` (integration, requires a running Docker daemon) succeeds for any package you touched
- `npm test` inside `dashboard/` succeeds for any frontend change

Do not push, open a PR, or claim work is "done" until these pass locally. CI runs the same jobs (`Go unit tests`, `Go integration tests`, `Dashboard tests`, `Lint`, plus a Playwright e2e smoke test) in `.github/workflows/ci.yml`; the `images` and `binaries` jobs in the same workflow (ghcr images + release binaries) gate on them. If you find yourself bypassing the loop ("I'll just push and let CI catch it"), stop and fix the loop first. A broken local feedback loop is a higher-priority bug than whatever you were working on.
</feedback_loop>

<supabase_js_compat>
**Instancez must stay wire-compatible with `@supabase/supabase-js`.** This is a load-bearing product promise, not a nice-to-have.

The contract is enforced by `internal/adapter/http/supabase_integration_test.go` (`TestSupabaseJSCompat`), which spins up Postgres in a container, boots the real instancez HTTP handler, then shells out to the Node harness in `test/integration/supabase-js/run.mjs` to drive `@supabase/supabase-js` against it. Any change touching auth, REST/PostgREST behavior, RPC, storage endpoints, error shapes, headers, or JWT/role handling MUST keep this test green.

Concrete rules that fall out of this:
- JWT `role` claim values are Supabase wire tokens (`anon`, `authenticated`, `service_role`) and must not be renamed even though the Postgres role names are configurable. See `internal/domain/database.go`.
- The `apikey` header, `Authorization: Bearer …`, PostgREST query operators (`eq`, `gt`, `like`, `order`, `limit`, embeds, `on_conflict`, `Prefer: return=…`, `Range`, etc.), error envelope shape, and `/auth/v1`, `/rest/v1`, `/storage/v1` URL prefixes are part of the contract.
- When adding a new feature exposed over HTTP, add coverage in `run.mjs` if `supabase-js` has a corresponding API surface. Don't ship behavior that only the bespoke Go tests exercise.
</supabase_js_compat>

## Common commands

<commands>
**Build & run:**
```sh
go build -o inz ./cmd/inz
./inz dev              # hot-reload dev server (requires INSTANCEZ_DATABASE_URL, a superuser DSN, to provision roles on every startup; or set INSTANCEZ_OWNER_DATABASE_URL and INSTANCEZ_AUTH_DATABASE_URL directly. JWT keys are DB-managed via auth.jwt_keys)
./inz serve            # production mode
./inz validate         # YAML syntax check + function file existence, no DB
./inz validate --use-dsn <owner-dsn>   # also prints migration plan (plan only, never applied)
./inz bundle           # build instancez.yaml + functions/ into a tar.gz artifact
./inz bundle --output s3://bucket/key  # build + upload to S3, print pointer
./inz serve --bundle s3://bucket/key#etag   # production mode using bundle (config + functions from single archive)
./inz serve --bundle s3://bucket/key --migrate --watch  # same, with migrations + hot-reload on ETag change
docker compose -f docker-compose.dev.yaml up   # full stack: postgres + backend + dashboard
```

**Go tests:**
```sh
go test -race ./...                                    # unit
go test -tags=integration -race ./...                  # full integration (needs Docker)
go test -run TestSupabaseJSCompat -tags=integration -race ./internal/adapter/http/...   # the supabase-js compat suite
go test -run <RegExp> ./internal/...                   # single test
```
Integration tests are gated behind `//go:build integration` and use `internal/testutil/dbboot` to provision `instancez_owner` + `authenticator` roles inside a fresh testcontainers Postgres. They need `docker` on `PATH`; the supabase-js harness additionally needs `node` + `npm`.

**Dashboard (in `dashboard/`):**
```sh
npm ci
npm test          # vitest run
npm run test:watch
npm run dev       # vite, port 5173
npm run build     # tsc -b && vite build
```
</commands>

## Architecture (the parts that span files)

<architecture>
Hexagonal layout under `internal/`:

```
cmd/inz/main.go
        │
        ▼
internal/cli/         cobra commands (dev, serve, init, validate, deploy, doctor, status, login, …)
        │
        ▼
internal/app/         engine.go orchestrates lifecycle: migrate → http + watcher
        │             — depends only on internal/domain interfaces
        ▼
internal/domain/      pure types + port interfaces (OwnerDB, RequestDB, Roles, Config, …)
        ▲
        │ implemented by
internal/adapter/     postgres (pgx pool), http (Gin handlers + PostgREST surface),
                      s3, resend
```

**Two Postgres logins, by design.** This is non-obvious and load-bearing:
- `INSTANCEZ_OWNER_DATABASE_URL` → privileged login (`CREATEROLE CREATEDB BYPASSRLS REPLICATION`). Used for migrations and DDL. Lives behind `domain.OwnerDB`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [instancez/instancez](https://github.com/instancez/instancez) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
