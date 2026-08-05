---
trigger: always_on
description: Guidance for AI agents working in the `sigma` repository.
---

# AGENTS.md

Guidance for AI agents working in the `sigma` repository.

## Project Overview

`sigma` is an OCI artifact storage and distribution system written in Go. It implements the [OCI Distribution Specification 1.1](https://github.com/opencontainers/distribution-spec/tree/v1.1.0) and acts as a private/public container registry. It supports docker registry v2 protocol, OCI images/artifacts/sboms, security scanning, registry proxy, namespace quotas, garbage collection, image signing (cosign), and image building via docker/podman/kubernetes.

- Module path: `github.com/go-sigma/sigma`
- Go version: 1.26.3 (godebug default=go1.26)
- License: Apache 2.0
- Entry point: `main.go` -> `cmd.Execute()`
- Default port: 3000
- Default credentials: `sigma` / `Admin@123`

## Build & Run

```bash
# Build the binary (requires CGO_ENABLED=1 for sqlite)
make build

# Run via docker
docker run --name sigma -p 3000:3000 --rm tosone/sigma:nightly-alpine

# Run locally (needs a config file, default /etc/sigma/config.yaml)
go run . server -c conf/config.yaml

# Subcommands: server, worker, builder, distribution, tools
```

Build tags required for tests/lint/build:

```text
netgo,timetzdata,exclude_graphdriver_btrfs,containers_image_openpgp
```

## Common Commands

```bash
# Lint
make lint                       # runs golangci-lint + hadolint
make lint-go                    # golangci-lint only (timeout 10m)

# Tests (set CI_DATABASE_TYPE=sqlite3|mysql|postgresql)
CI_DATABASE_TYPE=sqlite3 go test -parallel 1 -failfast \
  -tags "netgo,timetzdata,exclude_graphdriver_btrfs,containers_image_openpgp" \
  -timeout 30m ./...

# Code generation
make gormgen                    # regenerate gorm models/queries from DB
make swagen                     # regenerate swagger docs
make migration-create MIGRATION_NAME=<name>   # new SQL migration

# Formatting
make sql-format                 # format all SQL migrations
make addlicense                 # add Apache license headers

# Vendor / clean
make vendor
make clean
```

## Git Commit Messages

- Generated commit messages must use Conventional Commits and be written
  entirely in English, including the subject, body, footer, and verification
  notes.
- Include a body with what changed and why when the change needs more than a
  one-line subject, and put executed validation commands under `Verification:`.

## Test Conventions

- Tests use `github.com/stretchr/testify/require` (preferred over `assert`).
- For DI setup in tests, construct a fresh `dig.New()` container and provide `config.Configuration{}` plus `testkit.NewGin` before calling `factory{}.Initialize(digCon)`.
- Use `testkit.NewGin()` (in `pkg/testkit/gin.go`) to get a gin engine in `TestMode`; do not call `gin.New()` directly in tests.
- Repository tests must import the target repository domain package with a
  semantic alias, such as `registryrepo`, `namespacerepo`, or `userrepo`.
- Generate cryptographic keys (e.g., Ed25519) at runtime in tests; do not hardcode base64 keys.
- Tests are excluded for these packages (do not expect tests there): `pkg/testkit`, `pkg/dal/query`, `pkg/dal/cmd`, `pkg/handlers/apidocs`, and various `mocks` subpackages.
- Some storage tests (`pkg/storage/cos`, `pkg/storage/oss`) are skipped on PRs and require secret env vars (`COS_*`, `OSS_*`).
- Mocks are generated via `go.uber.org/mock` and live alongside the code as `*_mocks.go`.

## Code Conventions

### Import aliases

- Repository package aliases use the `repo` prefix followed by the domain name,
  such as `repouser`, `reponamespace`, `reporegistry`, and `repowebhook`.
- Service package aliases use the `svc` prefix followed by the domain name,
  such as `svcnamespace`, `svcrepository`, `svcmanifest`, and `svcaudit`.
- Do not use suffix-style aliases such as `userrepo`, `namespacesvc`, or
  `auditrepo`.

### Error handling

- Use `errcode.AsType[T](err)` (in `pkg/server/errcode/astype.go`) instead of `errors.As(err, &e)` for typed error assertions. The canonical usage: `errcode.AsType[errcode.ErrCode](err)`.
- Handlers return `error` (echo-style) and are wrapped via `server.Wrap(h)` or `server.WrapHandle(r, method, path, h)`. See `pkg/server/adapter.go`.
- Return `errcode.NewHTTPError(c, ...)` from handlers for structured HTTP errors.

### Dependency Injection

- DI is powered by `go.uber.org/dig`. Handlers declare `dig.In` and request services by field.
- Do NOT use the old `utils.GetObjFromDigCon` / `utils.MustGetObjFromDigCon` helpers (removed). Call `digCon.Invoke(func(deps ...) error { ... })` directly.
- Handler factories register routes via `handlers.Routers.Register(path.Base(reflect.TypeFor[factory]().PkgPath()), &factory{})` in an `init()`.

### Handler structure

- Each handler package (e.g. `pkg/server/handlers/systems`) defines a `Handler` interface, a private `handler` struct embedding `dig.In`, and a `factory{}` with `Initialize(dig.Container) error`.
- The `systems` handler uses field `SystemsService` of type `service.SystemsServiceFactory`. Do not use a field named `config`.
- Routes are grouped under `consts.APIV1` (i.e. `/api/v1`).

### IDs & crypto

- All application IDs must be time-ordered UUIDv7 strings.
- Generate UUIDs through `pkg/utils/uuid.NewV7String()` instead of calling an
  external UUID package directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-sigma/sigma](https://github.com/go-sigma/sigma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
