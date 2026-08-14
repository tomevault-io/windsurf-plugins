---
trigger: always_on
description: Generates type-safe per-table packages from database schema:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

```bash
# Run all tests (workspace root is NOT a Go module, so "..." wildcards do not
# expand from the repo root — list module paths explicitly, as below).
# All _test.go files live under _test/; the library paths below (aifei, config,
# …) carry no tests and are listed only to compile-check them.
go test ./aifei ./config ./dami ./db ./enjoy ./http ./json ./log ./nami ./server \
         ./tools/generator ./tools/damigen \
         ./plugins/cache ./plugins/dami ./plugins/dataisolate ./plugins/kafka ./plugins/nacos ./plugins/storage ./plugins/swagger ./plugins/elasticsearch ./plugins/xxljob \
         ./_test/json_test ./_test/log_test ./_test/config_test ./_test/dami_test \
         ./_test/server_test ./_test/nami_test ./_test/storage_test ./_test/swagger_test \
         ./_test/nacos_test ./_test/damigen_test ./_test/generator_test \
         ./_test/dataisolate_test ./_test/db_test ./_test/cache_test ./_test/kafka_test ./_test/enjoy_test

# Run tests for a single area (tests live under _test/<area>_test)
go test ./_test/db_test
go test ./_test/enjoy_test
go test ./_test/json_test
go test ./_test/log_test
go test ./_test/nacos_test
go test ./_test/generator_test
go test ./_test/config_test
go test ./_test/dami_test

# Run db integration tests (requires sqlite)
go test ./_test/db_test

# Run cache redis integration tests (embedded miniredis; no external redis)
go test ./_test/cache_test

# Run kafka integration tests (embedded franz-go kfake broker; no external kafka)
go test ./_test/kafka_test

# Run a single test
go test ./enjoy -run TestOutputExpr

# Run the demo
go run ./_test/demo
```

## Testing Conventions

**All test code lives under `_test/` — never co-locate `_test.go` files inside the library packages** (do not add tests to `db/`, `enjoy/`, `db/sql/`, etc.). When writing a new test, add it to the matching `_test/<area>_test/` module; if no module exists for that area, create one following the rules below. This is why the `db/sql` parser and `SqlKit` unit tests live in `_test/db_test`, imported as `dbsql "github.com/crazy-airhead/aifei-go/db/sql"` rather than as a `package sql` test inside `db/sql/`.

Each test area is its own Go module:

| Module | Covers | Embedded test dependency |
|--------|--------|--------------------------|
| `_test/db_test` | `db` + `db/sql` (CRUD, pagination, transactions, batch, Enjoy SQL directives, SQL parser) | `modernc.org/sqlite` |
| `_test/cache_test` | `plugins/cache` (local + Redis two-level cache) | `miniredis` |
| `_test/kafka_test` | `plugins/kafka` (producer/consumer, at-least-once) | franz-go `kfake` broker |
| `_test/enjoy_test` | `enjoy` template engine (black-box) | — |
| `_test/json_test` | `json` marshal/unmarshal wrappers (black-box) | — |
| `_test/log_test` | `log` logger interface + levels (black-box) | — |
| `_test/config_test` | `config` layered loading, `Props`/`Store`/`Sub`/`Bind` (black-box) | — |
| `_test/dami_test` | `dami` event bus (send/call/stream/lpc) + `plugins/dami` (black-box) | — |
| `_test/server_test` | `server` bootstrap: `In`/`Out`, `IoHandler`, middleware (black-box) | — |
| `_test/nami_test` | `nami` RPC client + `util`/`coder/json`/`channel/http` subpackages (black-box) | — |
| `_test/storage_test` | `plugins/storage` local + S3 clients, `Manager` (black-box) | `minio-go` |
| `_test/swagger_test` | `plugins/swagger` config loading (black-box) | — |
| `_test/dataisolate_test` | `plugins/dataisolate` tenant/row/column isolation (rewriter unit tests + sqlite integration) (black-box) | `modernc.org/sqlite`, `github.com/ajitpratap0/GoSQLX` |
| `_test/nacos_test` | `plugins/nacos` `NamiUpstream` (black-box) | — |
| `_test/damigen_test` | `tools/damigen` dami-provider codegen (black-box) | — |
| `_test/generator_test` | `tools/generator` schema→code (`MetaReader`, type mapping) (black-box) | `modernc.org/sqlite` |
| `_test/demo` | full demo app (run with `go run`, not a test suite) | `modernc.org/sqlite` |

Rules for adding tests:

- **External test package:** declare `package <area>_test` (matching the directory name) and test the **exported API only**. Import the library normally; alias sub-packages when convenient (e.g. `dbsql "github.com/crazy-airhead/aifei-go/db/sql"`). If you need to exercise unexported behavior, expose a test hook or assert via observable output/SQLite — do not drop a `package <lib>` test file into the library package.
- **Own module per area:** `module github.com/crazy-airhead/aifei-go/_test/<area>_test`, with `replace` directives pointing at the local modules it needs (`../../db`, `../../enjoy`, …), and add `use ./_test/<area>_test` to `go.work`.
- **Self-contained:** prefer embedded/in-process dependencies so `go test` needs no external services — SQLite via `modernc.org/sqlite`, `miniredis` for Redis, franz-go `kfake` for Kafka.
- **Issue regressions:** name the file `issueNNNN_test.go` inside the relevant `_test/<area>_test/` module, matching the `docs/issues/NNNN-*.md` note number.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crazy-airhead/aifei-go](https://github.com/crazy-airhead/aifei-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
