---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Stackdome — self-hosted PaaS that deploys/manages workloads across multiple Kubernetes clusters (Heroku-like: REST API, web UI, managed Postgres, git builds). Go API server + embedded React SPA.

## Architecture

**Hub-and-spoke.** Single API Server (hub) holds state in PostgreSQL and exposes REST + UI. Each managed cluster runs a `stackdome-agent` operator (spoke, Kubebuilder) that reconciles Custom Resources into K8s workloads.

- **Write path:** User → API Server → writes a Kubernetes CR → Cluster Agent → K8s resources (Deployment/Service/Ingress).
- **Status path:** Agent updates CR status → `pkg/controllers/*` controller-runtime watchers observe → PostgreSQL updated → UI/API reflect state.

Backend layering (`pkg/`): `handlers` (HTTP) → `services` (business logic) → `stores`/`pgstore` (DB). `presenters` convert model↔API. `builders` construct K8s objects. `clustermanager` multiplexes per-cluster connections. `worker/` does async reconciliation (managed by `workermanager`). `resourceaccess` = Casbin RBAC. `auth` = JWT/cookie. `validator/` is per-resource-type input validation. Entrypoints in `cmd/` (`servecmd`, `migratecmd`, `server` = routes/middleware).

The REST contract is `config/openapi/stackdome_api.yaml`. Go client (`pkg/api/openapi`) and frontend types/zod clients are **generated** from it — edit the spec, not generated code.

Frontend (`frontend/`, React 19 + Vite + Tailwind v4 + Radix) builds into `pkg/web/dist/` and is `go:embed`-ed into the server binary — no separate frontend deploy.

## Commands

Build system is **Mage** (`magefile.go`, namespaced targets); `Makefile` wraps some. Use `mage` for dev.

| Task | Command |
|------|---------|
| Bootstrap local env (Postgres + Kind + RBAC) | `mage dev:setup` |
| DB migrations | `mage migrate` |
| Build + run API server | `mage run` |
| Tear down local env | `mage dev:teardown` |
| Backend unit tests | `mage test:unit` (or `make test`) |
| Single Go test | `go test ./pkg/<pkg>/ -run TestName` |
| Integration tests (needs Docker/Kind) | `mage test:integration` |
| Lint Go | `golangci-lint run ./...` (or `mage lint`) |
| Format Go | `mage fmt` |
| Regenerate mocks | `make mocks` (mockgen via `go:generate`; add new packages to the target) |
| Regenerate OpenAPI Go client | `make generate` |
| Build frontend only | `make frontend` |
| Frontend dev / test / lint | `pnpm --prefix frontend dev` / `test` / `lint` |
| Frontend OpenAPI types/zod | `pnpm --prefix frontend generate:openapi-types` / `:openapi-zod` |

Notes:
- `mage dev:setup` is idempotent; reads/writes `.env` (creates from `.env_template`), writes cluster creds to `dev_env.yaml`.
- `mage build` skips `tsc -b` so unrelated frontend type errors don't block the binary; frontend's own `pnpm build` does run `tsc -b`.
- Integration tests need `TEST_KUBECONFIG` pointing at a Mage-created Kind cluster (`mage cluster:setup` / `cluster:delete`); state cached in `~/.cache/stackdome-api-server/clusters/`.
- New DB migration: scaffold with `hack/create_migration.sh`; ordered files live in `pkg/db/migrations/`.
- Full end-to-end demo env: `hack/run_local.sh [stack.json]`.
- **Tests use Ginkgo.** All Go tests are written with Ginkgo v2 + Gomega (`Describe`/`It`/`Expect`), never bare `testing.T` test functions. Ginkgo allows exactly one `RunSpecs` per package — if a suite bootstrap (`*_suite_test.go` or any `RunSpecs` call) already exists, add `var _ = Describe(...)` blocks to new files instead of a second suite. Existing bare-`testing.T` tests may stay, but new tests never add to them.
- **Mocks:** Always use `go.uber.org/mock/gomock` + `mockgen`. Never hand-roll mock structs. For package-private interfaces, generate mocks in-package with `mockgen -source=<file>.go -destination=<file>_mock.go -package=<pkg>` and add a `//go:generate` directive to the source file. Generated mock files do NOT take a `_test` suffix — `<file>_mock.go`, never `<file>_mock_test.go`. Existing generated mocks live in `pkg/mocks/` (for exported interfaces) and `*_mock.go` files (for in-package interfaces); older `*_mock_test.go` files may be renamed when their package is next touched.
- **No magic strings.** Always use defined constants (model enums, annotation keys, state values, error codes) instead of raw string literals. This applies to both production code and tests — e.g. use `models.ReleaseStateReleased` not `"Released"`, `models.SkipClusterProvisioningAnnotation` not `"stack.stackdome.io/skip-cluster-provisioning"`. If no constant exists and one is needed, define it first. If a raw string is truly unavoidable, get explicit approval before using it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stackdome/Stackdome](https://github.com/Stackdome/Stackdome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
