---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Kuberpult

Kuberpult is a Kubernetes deployment management system that manages *what gets deployed next* across cluster environments (dev, staging, production). It complements ArgoCD (which applies current versions). Think of it as a "catapult" for rolling out microservices.

## Architecture

Five Go microservices plus shared packages:

- **cd-service** — Core logic: release management, deployment, locks, release trains. Exposes gRPC only (HTTP port serves only a `/health` check).
- **manifest-repo-export-service** — Reads DB state, pushes manifest changes to the git repository that ArgoCD watches.
- **frontend-service** — REST/gRPC-web adapter backing the React web UI.
- **rollout-service** — Coordinates direct connection to Argo CD via grpc.
- **reposerver-service** — Manages repository synchronization, implements Argo CD grpc endpoints.
- **pkg/** — Shared packages: `api/` (protobuf), `db/`, `auth/`, `logger/`, `metrics/`, `tracing/`, `testutil/`, `migrations/`.

Services communicate via gRPC (port 8443 internal, 8080 HTTP). All service config comes from environment variables. PostgreSQL is the only supported database; migrations run automatically on startup.

## Build & Run

All build targets use Docker + the builder image. The builder image must be built locally before anything else:

```bash
make builder          # Build local builder image (required first time)
make kuberpult        # Build everything and start all services via docker-compose
make kuberpult-freshdb  # Same but with a clean database
make reset-db         # Delete the postgres volume
```

Build a single service Docker image:
```bash
IMAGE_TAG=local make -C services/cd-service docker
```

## Docker Concept
Our "builder" Dockerfile contains the go.mod and runs go mod download.
Other Dockerfiles that depend on it, should not use go mod download again. They just copy from the builder.

## Testing

Tests run inside Docker using the builder image and connect to a test PostgreSQL instance.
Most tests require the DB.

```bash
make test                          # Run all tests (all services + pkg)
make -C services/cd-service test   # Test a single service
make -C pkg test                   # Test shared packages
make -C services/manifest-repo-export-service test GO_TEST_ARGS="-run TestParseEnvironmentOverrides -v" # Run specific test verbose
```

Do not try to run `go test` directly - it will likely fail, either because of the Database, or because of libgit2 issues.


Frontend tests (inside `services/frontend-service`):
```bash
pnpm test              # Watch mode
pnpm test-ci           # CI mode (no watch)
```


## Linting

Go linting runs via golangci-lint inside Docker:
```bash
make -C services/cd-service lint      # Lint a service
make -C services/cd-service lint-fix  # Auto-fix
```

Enabled linters: `errcheck`, `govet`, `ineffassign`, `unused`, `asciicheck`, `bodyclose`, `copyloopvar`, `staticcheck`, `unconvert`, `gocritic` (importShadow check), `revive` (redundant-import-alias). Generated code in `pkg/publicapi/` is excluded.

Frontend linting (inside `services/frontend-service`):
```bash
pnpm lint
pnpm lint-fix
```

## Code Coverage Thresholds

Coverage is enforced at test time and the build fails if thresholds are not met.
The coverage thresholds are defined in each service's `services/<service-name>/Makefile`, look for `MIN_COVERAGE`.

## Go Test Patterns

All Go tests must follow these conventions:

- Even for the simplest test, immediately create a "table" (go slice) so that testing different variations is easy in the future.
- In a table-driven test, only put the really relevant parts into the table data. Data that is identical for all cases should not be part of the table.
- Don't use function-typed fields in the table struct (e.g. `fn func() error`). Keep all logic inline in the test loop body.
- Omit the line `tc := tc` at the beginning of test loops — it is outdated (Go 1.22+ handles loop variable capture correctly).

**Table-driven tests:**
```go
tcs := []struct {
    Name string
    // ...
}{
    {Name: "happy path", ...},
}
for _, tc := range tcs {
    t.Run(tc.Name, func(t *testing.T) { ... })
}
```

**Assertions with `cmpDiff`:**
```go
if diff := cmpDiff(expected, actual); diff != "" {
    t.Errorf("mismatch (-want, +got):\n%s", diff)
}
```
Always use cmpDiff, never use cmp.Diff, as it is not type-safe.

**Proto message comparison:**
```go
if diff := cmpDiff(expected, actual, protocmp.Transform()); diff != "" {
    t.Errorf("mismatch (-want, +got):\n%s", diff)
}
```

Do not compare raw JSON strings; always compare Go objects.

## Commit Conventions

This repo uses conventional commits enforced by commitlint:
- `fix:` → PATCH bump
- `feat:` → MINOR bump
- `feat!:` / `fix!:` → MAJOR bump (breaking change)

Types `revert`, `perf`, `docs`, `test`, `refactor`, `style`, `chore`, `build`, `ci` are restricted (not allowed).

## Protobuf / API Generation

Proto definitions live in `pkg/api/`. Regenerate after changes:
```bash
make -C pkg gen
```

For frontend TypeScript types:
```bash
make -C services/frontend-service gen-api
```

## Helm Chart


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freiheit-com/kuberpult](https://github.com/freiheit-com/kuberpult) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
