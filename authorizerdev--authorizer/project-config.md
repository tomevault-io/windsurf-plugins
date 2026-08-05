---
trigger: always_on
description: Project instructions for AI coding tools (Claude Code, Cursor, Gemini, Copilot).
---

# AGENTS.md

Project instructions for AI coding tools (Claude Code, Cursor, Gemini, Copilot).

Detailed conventions live in `.claude/skills/authorizer-*/SKILL.md` and load on demand when matching files are touched. Role-specific expertise lives in `.claude/agents/`.

## What is Authorizer

Open-source, self-hosted authentication and authorization server. Supports 13+ databases, OAuth2/OIDC, social logins, MFA, magic links, role-based access, webhooks, and email templating.

**Stack**: Go 1.24+, Gin, gqlgen, GORM, zerolog, Cobra CLI, JWT, OAuth2/OIDC, gRPC (+ grpc-gateway REST), buf/protobuf.
**v2**: CLI flags for all config — no `.env` or OS env vars.

## Make Commands

### Dev & build

```bash
make dev                  # Run server locally (SQLite, embedded dev RSA keys)
make build                # Cross-compile server binary → build/{os}/{arch}/authorizer
make build-app            # Build login UI (web/app)
make build-dashboard      # Build admin UI (web/dashboard)
make all                  # build + build-app + build-dashboard
make bootstrap            # Install gox (required by make build)
make clean                # Remove build/
```

### Docker

```bash
make build-local-image    # docker build (IMAGE defaults to quay.io/authorizer/authorizer:$(VERSION))
make build-push-image     # Multi-arch buildx push
make trivy-scan           # Scan Docker image for HIGH/CRITICAL CVEs (IMAGE= override)
```

`VERSION` defaults to `0.1.0-local`; override with `make build VERSION=1.2.3`.

### Code generation

```bash
make generate-graphql     # Regenerate gqlgen output after schema.graphqls change; runs go mod tidy
make generate-db-template # Scaffold new storage provider: make generate-db-template dbname=foo

make proto-gen            # buf generate → gen/ (installs buf if missing)
make proto-lint           # buf lint on proto/
make proto-breaking       # Breaking-change check vs origin/main (override: BUF_BREAKING_AGAINST)
make proto-check          # proto-gen + fail if gen/ is stale (CI)
make proto-tools          # Install buf only
```

After editing `internal/graph/schema.graphqls` → `make generate-graphql`.
After editing `proto/` → `make proto-gen` and commit `gen/`.

### Format & lint

```bash
make fmt                  # fmt-go + fmt-ts
make fmt-go               # gofmt -s (excludes gen/)
make fmt-ts               # Prettier on web/app and web/dashboard

make lint                 # lint-go + lint-ts
make lint-go              # golangci-lint (installs if missing; excludes gen/ via .golangci.yml)
make lint-ts              # Prettier --check on both web apps
make lint-tools           # Install golangci-lint only
```

Run `make fmt` before committing; CI runs `make lint`.

### Tests

```bash
make test                 # Full module test run; TEST_DBS=sqlite (integration tests always SQLite)
make test-sqlite          # Same as test — explicit SQLite-only, no Docker
make test-all-db          # All 7 DBs via Docker (postgres, sqlite, mongodb, arangodb, scylladb, dynamodb, couchbase)
make smoke                # Release e2e smoke tests (build tag `smoke`, 5m timeout)

# Single-DB targets (each spins up Docker, runs tests, tears down)
make test-postgres
make test-mongodb
make test-scylladb
make test-arangodb
make test-dynamodb
make test-couchbase

# Docker helpers for test-all-db
make test-docker-up       # Start all test DB containers + Redis
make test-cleanup         # Remove all test containers
make test-cleanup-postgres | test-cleanup-mongodb | test-cleanup-scylladb
make test-cleanup-arangodb | test-cleanup-dynamodb | test-cleanup-couchbase
```

**Test env vars**:
- `TEST_DBS` — comma-separated list for storage provider tests (e.g. `sqlite`, `postgres,mongodb`). Defaults to all when unset in storage tests.
- `TEST_ENABLE_REDIS=1` — include Redis memory_store tests (skipped by default).

**Single test** (integration tests use SQLite via `getTestConfig()`):

```bash
go clean --testcache && TEST_DBS="sqlite" go test -p 1 -v -run TestSignup ./internal/integration_tests/
```

Use `-p 1` for integration tests (shared state). Storage tests honour `TEST_DBS`.

**Verifying a change before calling it done** — run these, in order, and only report success after they actually pass in this session:

1. `go build ./...` — compiles.
2. `go vet ./...` — catches suspicious constructs (nil-deref-prone patterns, unreachable code, struct-copy issues).
3. `make test` (or the single-test command above for a targeted case) — SQLite integration suite passes.
4. **Storage-layer changes only**: also run at least one non-SQL backend (`make test-mongodb`, `make test-arangodb`, etc.) or `make test-all-db` — a fix verified only on SQLite has not verified cross-DB parity, and this repo's whole storage layer is defined by that parity holding.
5. `make lint` — must be clean before opening a PR; CI enforces it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [authorizerdev/authorizer](https://github.com/authorizerdev/authorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
