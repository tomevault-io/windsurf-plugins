---
trigger: always_on
description: OpenMeter is a usage metering and billing platform for AI and DevTool companies, built in Go.
---

# OpenMeter

OpenMeter is a usage metering and billing platform for AI and DevTool companies, built in Go.

## Quick Reference

Use the `Makefile` for all common tasks. A `justfile` also exists but is seldom used.
OpenMeter is a metering and billing platform with usage based pricing and access control.

## Tips for working with the codebase

If during your work anything confuses you or something isn't trivial for you, please augment AGENTS.md with your findings so next time it will be easier for you. AGENTS.md files are for you to edit and update as you go so you can interact with the codebase the most effectively.

Development commands are run via `Makefile`, it contains all commonly used commands during development. A `justfile` is also present but seldom used. Use the Makefile commands for common tasks like running tests, generating code, linting, etc.
The committed `.nvmrc` is the GitHub Actions source of truth for Node-based jobs on GitHub-hosted runners. Keep it aligned with the Nix `.#ci` shell's `node -v`; `flake.nix` refreshes it in `enterShell`, and CI validates the file against the Nix shell before running builds.

## AGENTS.md maintenance

- Treat this file as long-lived project guidance for all agents and contributors.
- Treat AGENTS.md as the repo-local source of truth. Do not bypass coding style, workflow, testing, or documentation guidance in this file. If a requested change appears to conflict with AGENTS.md, ask the human developer/reviewer to confirm the exception before proceeding, and make the exception explicit in the handoff.
- Prefer durable wording over time-based wording (avoid labels like "recent", "latest", "today").
- Keep entries actionable and specific (what to do, where, and why), not conversational history.
- Capture universal truths and cross-cutting coding conventions here when they become repeated practice or reviewer expectation. Do not leave them only in chat or pull request comments.
- Capture subsystem-specific guidance in the closest applicable nested `AGENTS.md` when the guidance should always apply to a subtree, such as `api/spec/AGENTS.md` for TypeSpec and SDK guidance. Use skills for reusable workflows or domain procedures that agents opt into for a task. Skills must stay usable by both Claude and Codex: write them as plain repo guidance, keep `.agents/skills` as the source of truth, and avoid assistant-specific assumptions unless a workflow truly requires them.
- When adding new guidance, fold it into the most relevant section and remove/merge stale or duplicate notes.

## Testing

| Task | Command |
|------|---------|
| Start dependencies | `make up` |
| Stop dependencies | `make down` |
| Run API server (hot reload) | `make server` |
| Run all tests | `make test` (root module only; excludes `e2e/`, its own module) |
| Run e2e tests | `make etoe` |
| Generate all code | `make generate-all` |
| Generate Go code only | `make generate` (runs `go generate ./...`) |
| Generate API + SDKs | `make gen-api` |
| Lint all | `make lint` |
| Lint Go only | `make lint-go` |
| Format code | `make fmt` |
| Tidy modules | `make mod` (root + `collector` + `e2e`) |
| Build all binaries | `make build` |

## Architecture

**Entry points:** `cmd/server`, `cmd/billing-worker`, `cmd/balance-worker`, `cmd/sink-worker`, `cmd/notification-service`, `cmd/jobs`

Core business logic is in `openmeter/`, shared utilities in `pkg/`, API layer in `api/`.

**Stack:** Go + PostgreSQL (Ent ORM) + Kafka + ClickHouse. API defined in TypeSpec, generated to OpenAPI.

Domain packages under `openmeter/` follow a layered service/adapter pattern. See the `/service` skill for full details.

`cmd/server/main.go` now migrates the database before creating the default namespace. Register namespace handlers before `initNamespace(...)` if they must provision the default namespace during startup.

**Module layout:** the repo is three separate Go modules. The root module (`github.com/openmeterio/openmeter`) holds all production code (`cmd/`, `openmeter/`, `pkg/`, etc.). `api/v3/client` is the standalone, publishable v3 Go SDK module. `e2e/` is a third, never-published, test-only module that imports both — it pins itself to the working tree of each via `replace github.com/openmeterio/openmeter => ../` and `replace .../api/v3/client => ../api/v3/client`, so e2e always tests local code regardless of what's tagged. The root module must never `require` the SDK module: a `require` on an untagged nested module resolves to an unresolvable `v0.0.0` for anyone outside this repo (the `replace` directive that makes it resolve locally is invisible downstream), so any code that needs the SDK — today, only `e2e/` — has to live in its own module rather than the root one. Because of this, root `go build ./...` / `go test ./...` / `go vet ./...` no longer see `e2e/` at all; use `make etoe` (runs it against a live server) or `go test -C e2e ./...` / `go vet -C e2e ./...` (compiles it standalone, no server needed) instead. `make lint-go` and `make mod` already cover all three modules. For editor/gopls support across all three, run `go work init . ./api/v3/client ./e2e` locally — `go.work`/`go.work.sum` are gitignored and must never be committed.

### Project Layout

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openmeterio/openmeter](https://github.com/openmeterio/openmeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
