---
trigger: always_on
description: Go clean-architecture reference service. Three domains (`user`, `task`, `translation`) exposed
---

# go-clean-template

Go clean-architecture reference service. Three domains (`user`, `task`, `translation`) exposed
over four transports (REST/Fiber, gRPC, RabbitMQ RPC, NATS RPC) from one shared use-case layer.
Module path: `github.com/evrone/go-clean-template`. The Go version is declared in `go.mod`.

## Commands — drive everything through the Makefile

The `Makefile` loads `.env` (falling back to `.env.example`) and exports it into every target, and
several targets carry flags that matter. Running the underlying tool directly means running it with
different settings than CI. Use the target, not the tool.

| Task | Use this | Not this |
| --- | --- | --- |
| Install tool binaries (`swag`, `mockgen`, `migrate`, linters) | `make bin-deps` | `go install ...` |
| Unit tests | `make test` | `go test ./...` — the target adds `-race -covermode atomic` and scopes to `./internal/... ./pkg/...` |
| Integration tests | `make compose-up-integration-test` | `make integration-test` — see below |
| Lint | `make linter-golangci` | `golangci-lint run` |
| Format | `make format` | `gofmt` — the target runs `go fix`, `gofumpt`, and `gci` with the repo's import grouping |
| Regenerate mocks | `make mock` | `mockgen ...` |
| Regenerate Swagger | `make swag-v1` | `swag init` — the target passes `--parseDependency -g internal/controller/restapi/router.go` |
| Regenerate protobuf | `make proto-v1` | `protoc ...` |
| Tidy / verify modules | `make deps` | `go mod tidy` |
| Vulnerability scan | `make deps-audit` | `govulncheck ./...` |
| Start dependencies (Postgres, RabbitMQ, NATS) | `make compose-up` | `docker compose up` |
| Start the whole stack including the app | `make compose-up-all` | `docker compose up` |
| Tear down | `make compose-down` | `docker compose down` |
| Run the app locally | `make run` | `go run ./cmd/app` — the target regenerates docs and builds with `-tags migrate` |
| Create a migration | `make migrate-create <name>` | `migrate create ...` |
| Apply migrations | `make migrate-up` | `migrate -path ... up` |
| Full check before pushing | `make pre-commit` | running the steps by hand |

`make help` lists every target.

Three traps in these targets:

- **`make integration-test` is not the one you want.** It runs `go test ./integration-test/...` on
  the host, where the suite cannot resolve the container hostnames it needs, so it always fails.
  `make compose-up-integration-test` is the real entry point.
- **`make migrate-create <name>` prints an error after it succeeds.** The target reads the name via
  `$(word 2,$(MAKECMDGOALS))`, so `make` then tries to build `<name>` as a target and reports
  `No rule to make target`. The migration files are already created; ignore that line.
- **`make run` and `make pre-commit` depend on `swag-v1` and `proto-v1`**, so they need `swag` and
  `protoc` on `PATH`. Run `make bin-deps` first (`protoc` itself is not installed by it).

Never claim a change is done without `make format`, `make linter-golangci` and `make test` passing.

## Dependency rule

```
cmd/app → internal/app → internal/controller/*  ─┐
                       → internal/repo/*        ─┤→ internal/usecase (interfaces)
                                                 └→ internal/entity
```

- `internal/entity` — domain types and sentinel errors. Imports nothing from this module.
- `internal/usecase/contracts.go` — the interfaces controllers call. `internal/usecase/<domain>/`
  implements them.
- `internal/repo/contracts.go` — the interfaces use cases call. `internal/repo/persistent/<domain>/`
  (Postgres) and `internal/repo/webapi/` (outbound HTTP) implement them.
- `internal/controller/<transport>/v1/` — one package per transport, each with its own
  `request/` and `response/` DTOs. Controllers never import each other.
- `pkg/` — transport-agnostic infrastructure (servers, logger, jwt, postgres, tracing). Must not
  import `internal/`.

Inner layers never import outer ones. Wiring happens exactly once, in `internal/app/app.go`
(`initUseCases`, `initServers`).

## Where code goes

Decide by asking what the code knows about:

| The code knows about… | It belongs in | Shape |
| --- | --- | --- |
| nothing but the domain | `internal/entity/<name>.go` | struct + methods (`Task.Transition`, `TaskStatus.Valid`) |
| a domain rule that spans repositories | `internal/usecase/<domain>/<domain>.go` | method on `UseCase` |
| SQL, a table, a driver error code | `internal/repo/persistent/<domain>/<domain>.go` | method on `Repo` |
| an outbound HTTP API | `internal/repo/webapi/<name>.go` | method on the webapi struct |
| an HTTP status, a gRPC code, a message envelope | `internal/controller/<transport>/v1/<domain>.go` | handler |
| a server, pool, client or middleware with no domain knowledge | `pkg/<name>/` | reusable package |

Layout rules:

- One file per domain per layer, named after the domain (`task.go`, `user.go`, `translation.go`),
  plus a `tracing.go` in every `usecase/<domain>/` and `repo/persistent/<domain>/` package.
- Package name matches the directory name. `internal/repo/persistent/translation/` is the one
  violation in the tree — it declares `package persistent`, which is why `internal/app/app.go`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evrone/go-clean-template](https://github.com/evrone/go-clean-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
