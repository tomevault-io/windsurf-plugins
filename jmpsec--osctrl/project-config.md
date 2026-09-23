---
trigger: always_on
description: This repository is a Go monorepo for `osctrl`. Entrypoints live under `cmd/`:
---

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Go monorepo for `osctrl`. Entrypoints live under `cmd/`:

- `cmd/tls`: `osctrl-tls`, the osquery-facing remote API endpoint
- `cmd/api`: `osctrl-api`, the REST API used by the React frontend, CLI, automation, and hosted MCP
- `cmd/cli`: `osctrl-cli`, the operator/bootstrap CLI
- `cmd/mcp`: `osctrl-mcp`, the standalone MCP stdio server

The operator UI is a React/Vite/TypeScript SPA under `frontend/`. It calls `osctrl-api`; there is no separate `osctrl-admin` service.

Shared domain and infrastructure packages live under `pkg/`, notably:

- `pkg/backend`, `pkg/cache`, `pkg/config`, `pkg/serviceconfig`, `pkg/servicecommands`
- `pkg/environments`, `pkg/nodes`, `pkg/queries`, `pkg/carves`, `pkg/posture`
- `pkg/console`, `pkg/fileexplorer`, `pkg/activity`, `pkg/alerts`
- `pkg/logging`, `pkg/logsinks`, `pkg/settings`, `pkg/tags`, `pkg/users`, `pkg/auditlog`
- `pkg/auth`, `pkg/authproviders`, `pkg/mfa`, `pkg/apiclient`, `pkg/mcp`

HTTP handlers live in `cmd/api/handlers` and `cmd/tls/handlers`. Deployment and release material lives under `deploy/`, `.github/workflows/`, and `.goreleaser.yml`; development helpers live under `tools/`.

Read [ARCHITECTURE.md](./ARCHITECTURE.md) before structural or cross-service changes. Read [MCP.md](./MCP.md) before changing MCP tools, transports, permissions, or write gating.

## Build, Test, and Development Commands

Use the root `Makefile` for standard workflows:

- `make build`: build the frontend and all four Go binaries; binaries are written to `bin/`
- `make tls`, `make api`, `make cli`, `make mcp`: build one Go component
- `make static`: build static TLS, API, CLI, and MCP binaries
- `make frontend-dev`: run the Vite development server
- `make frontend-test`: run frontend unit tests and TypeScript checking
- `make frontend-build`: type-check and build the production frontend bundle
- `make openapi`: regenerate Swagger/OpenAPI outputs
- `make openapi-check`: verify generated API documentation is current
- `make clean-dist`: remove `dist/` before a direct GoReleaser run
- `make release-check`: validate `.goreleaser.yml`
- `make release-build`: create a clean local snapshot release
- `make tidy`: clean generated Go dependency files and run `go mod tidy`

For focused validation:

- `go test ./...`: run all Go tests
- `go test ./cmd/api/... ./cmd/tls/...`: test service code and handlers
- `go test ./pkg/mcp ./pkg/apiclient`: test standalone/hosted MCP behavior and API delegation
- `go test ./pkg/<area>`: test one changed package
- `golangci-lint run ./...`: run Go lint checks when available
- `cd frontend && npm test`: run frontend unit tests
- `cd frontend && npm run check`: run TypeScript checks
- `cd frontend && npm run test:e2e`: run Playwright end-to-end tests

For local stack work:

- `docker compose -f docker-compose-dev.yml up -d`
- `make docker_dev_build`
- `make docker_dev_logs_tls`, `make docker_dev_logs_api`, `make docker_dev_logs_frontend`

`make docker_dev_build` requires a local `.env` and generated development certificates. Do not commit either. If lint or tests cannot write global caches, use writable cache paths such as `GOCACHE=/tmp/...` and `GOLANGCI_LINT_CACHE=/tmp/...`.

## Coding Style & Naming Conventions

Follow standard Go conventions:

- Format Go with `gofmt`
- Keep package names lowercase and exported identifiers in `CamelCase`
- Prefer table-driven tests
- Keep HTTP handlers thin and move reusable behavior into `pkg/*`
- Reuse existing managers, caches, readers, and typed configuration instead of duplicating persistence or protocol logic

For frontend work, follow the existing React Query, TanStack Router, Radix, Tailwind, and `lucide-react` patterns. Keep API contracts in `frontend/src/api` and colocate component tests with the feature being changed.

When changing HTTP behavior:

- Preserve `http.NewServeMux` and Go 1.22 method/path patterns
- Reuse `handlerAuthCheck` in `cmd/api/auth.go` for protected API routes
- Enforce environment permissions server-side with `h.Users.CheckPermissions`; UI visibility is not authorization
- Keep route registration in the owning service's `main.go`
- Apply existing request-size, trusted-proxy, CSRF, audit, and rate-limit patterns where relevant
- Regenerate and check OpenAPI output when public REST behavior changes

When changing MCP behavior:

- Keep shared tool definitions and validation in `pkg/mcp`
- Keep standalone access delegated through `pkg/apiclient`; it must not connect directly to the database
- Keep hosted MCP dispatching through existing API handlers so their authentication, authorization, and audit behavior remains authoritative
- Register mutating tools only behind the independent write opt-in; normal API permission checks still apply
- Treat node names, paths, process data, and query rows as untrusted model context

When changing persistence:

- Understand the owning GORM model and manager constructor first
- This repository uses startup `AutoMigrate` rather than ordered migrations, so model changes are production-impacting
- Do not assume database foreign keys or cascade behavior; many relationships are maintained in application code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmpsec/osctrl](https://github.com/jmpsec/osctrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
