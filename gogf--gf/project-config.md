---
trigger: always_on
description: This is the [GoFrame](https://goframe.org) framework (`github.com/gogf/gf/v2`) — a modular Go application framework. The repository is a **multi-module monorepo**: the root module hosts the core framework, while `cmd/gf` and every directory under `contrib/` are independent Go modules with their own `go.mod`.
---

# Repository Overview

This is the [GoFrame](https://goframe.org) framework (`github.com/gogf/gf/v2`) — a modular Go application framework. The repository is a **multi-module monorepo**: the root module hosts the core framework, while `cmd/gf` and every directory under `contrib/` are independent Go modules with their own `go.mod`.

- Root module: `github.com/gogf/gf/v2` (Go 1.23+)
- Tooling module: `cmd/gf` (CLI: `gf` command, separate `go.work`)
- Plugin modules under `contrib/`: `config/*`, `drivers/*` (mysql, pgsql, mssql, oracle, sqlite, clickhouse, dm, gaussdb, mariadb, oceanbase, tidb, sqlitecgo), `metric/*`, `nosql/*`, `registry/*`, `rpc/grpcx`, `sdk/*`, `trace/*`. Each is published as its own module so users only pull what they need.

The root framework intentionally has **minimal external dependencies** (see `go.mod`). Anything requiring a heavy third-party dep (a DB driver, a registry client, an RPC stack) lives in `contrib/`.

## Top-level package map

| Path | Purpose |
| --- | --- |
| `frame/g`, `frame/gins` | Convenience facade (`g.Server()`, `g.DB()`, `g.Cfg()`) and the singleton/instance container |
| `net/` | `ghttp` (HTTP server/router), `gclient` (HTTP client), `gtcp`, `gudp`, `gsel` (load balancing), `gsvc` (service discovery), `gtrace`, `goai` (OpenAPI) |
| `os/` | OS abstractions: `gcfg` (config), `gcmd` (CLI), `gcron`, `glog`, `gfile`, `gres` (resource embedding), `gview` (templating), `gcache`, `gsession`, `gctx`, `gtimer`, `gproc`, `gmetric`, `gfsnotify`, `gstructs` |
| `database/` | `gdb` (ORM/query builder, driver-agnostic core), `gredis` (Redis client core) |
| `container/` | Concurrent-safe data structures: `garray`, `gmap`, `gset`, `gtree`, `glist`, `gqueue`, `gring`, `gpool`, `gtype`, `gvar` |
| `encoding/` | Codecs for JSON/XML/YAML/TOML/INI/Properties, base64, charsets, compression, hashes, HTML, URL |
| `text/` | `gstr`, `gregex` |
| `util/` | `gconv` (universal conversion — heavily used), `gvalid` (validation), `gutil`, `grand`, `guid`, `gtag`, `gmeta`, `gpage`, `gmode` |
| `crypto/` | `gaes`, `gdes`, `grsa`, `gmd5`, `gsha*`, `gcrc32` |
| `errors/` | `gerror` (stack-aware errors), `gcode` (error code registry) |
| `internal/` | Framework-internal helpers (do not import from outside the root module) |
| `test/gtest` | The framework's own testing helpers — used throughout the test suite |

Concrete database drivers and Redis adapters live under `contrib/drivers/` and `contrib/nosql/`; the `database/gdb` and `database/gredis` packages define the abstract layer.

# Common Commands

All commands run from the repository root unless noted.

## Build / lint / tidy

```bash
# Tidy every go.mod in the repo (root, cmd/gf, contrib/*) — strips // indirect and toolchain lines
make tidy

# Run the project's golangci-lint config (.golangci.yml)
make lint
# Equivalent: golangci-lint run -c .golangci.yml
```

`make tidy` invokes `.make_tidy.sh`, which `cd`s into every directory containing a `go.mod` (skipping `testdata/` and `examples/`) and runs `go mod tidy`. After editing imports in any module, run this from the repo root.

## Tests

Each Go module must be tested from inside its own directory because they are separate modules. The CI script (`.github/workflows/scripts/ci-main.sh`) iterates every `go.mod`:

```bash
# Build + race-enabled tests for the root module
go build ./...
go test ./... -count=1 -race

# Coverage (matches CI 'coverage' mode)
go test ./... -count=1 -race -coverprofile=coverage.out -covermode=atomic \
  -coverpkg=./...,github.com/gogf/gf/...

# Run a single package's tests
go test -count=1 -race ./os/gcfg/...

# Run a single test by name
go test -count=1 -race -run TestCfg_Get ./os/gcfg/

# Test a contrib module — must cd in first (separate go.mod)
cd contrib/drivers/mysql && go test ./... -count=1 -race
```

Many tests (database drivers, registry clients, redis cluster, apollo/nacos config) require backing services. CI starts them via docker-compose files under `.github/workflows/{redis,apollo,nacos,consul}/`. Locally, use:

```bash
make docker                      # start the default local stack
make docker cmd=start svc=mysql  # start a specific service
make docker cmd=stop  svc=mysql
```

## CLI tool

```bash
cd cmd/gf && go run . <subcommand>   # iterate on the gf CLI itself
```

# Architecture Notes Worth Knowing Up Front

- **The `frame/g` package is a facade, not a library.** It re-exports types and provides singleton accessors (`g.DB()`, `g.Server()`, `g.Cfg()`, `g.Log()`) backed by `frame/gins`. Examples in docs use `g.*` heavily; framework-internal code generally imports the underlying packages directly.
- **`util/gconv` is foundational.** Most public APIs accept `any` and use `gconv` for type coercion. When changing argument handling, search for `gconv.` usage to understand the conversion contract.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gogf/gf](https://github.com/gogf/gf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
