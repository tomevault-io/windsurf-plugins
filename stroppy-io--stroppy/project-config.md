---
trigger: always_on
description: Database stress testing CLI powered by k6. Apache 2.0.
---

# Stroppy — Agent Context

Database stress testing CLI powered by k6. Apache 2.0.

## Binary Layout

- `./build/stroppy` — main binary (built by `make build`)
- `./build/k6` — k6 with stroppy extension embedded
- Both produced by the same `make build` target

## Build & Lint

```
make build          # ALWAYS use this — never go build ./...
make linter_fix     # run first, auto-fixes formatting
make linter         # read-only check after linter_fix
make tests          # all tests with race detector and coverage
make proto          # regenerate Go/TS/docs from .proto; wipes pkg/common/proto/* — never hand-edit generated files
make ts-test        # TypeScript unit tests
make ts-typecheck   # typecheck helpers.ts / datagen.ts / parse_sql.ts / stroppy.d.ts
```

**Embedded FS rebuild rule:** `workloads/` is `//go:embed *` — if you pass a workload by short name (`tpcc/tx`, `tpcb/procs`), the binary serves from its embedded snapshot. Edits to `workloads/` on disk have **no effect** until `make build` reruns.

**Local path bypass:** If you pass an explicit local path (`./workloads/tpcc/tx.ts`, `./workloads/tpcc/pg.sql`), the runner resolves from cwd **first** — no rebuild needed. Use this during the edit-run loop:
```bash
./build/stroppy run ./workloads/tpcc/tx.ts ./workloads/tpcc/pg.sql -d pg -D url=postgres://...
```

Resolution order: **cwd → `~/.stroppy/` → embedded**.

## Directory Map

| Path | Role |
|------|------|
| `cmd/stroppy/commands/` | cobra CLI subcommands: gen, run, probe, version |
| `cmd/xk6air/` | k6 extension entry; registers `k6/x/stroppy`, manages per-VU instances |
| `pkg/driver/dispatcher.go` | driver registry: `RegisterDriver()` + `Dispatch()` |
| `pkg/driver/{postgres,mysql,picodata,ydb,noop,csv}/` | driver implementations |
| `pkg/driver/sqldriver/` | shared sql.DB-backed base (mysql, ydb use this) |
| `pkg/datagen/` | relational data-generation runtime: compile, expr, runtime, lookup, cohort, stdlib, seed |
| `internal/static/` | `helpers.ts`, `datagen.ts`, `parse_sql.ts`, generated TS type bindings |
| `internal/runner/` | esbuild transpilation, config extraction via Sobek, k6 process management |
| `proto/stroppy/` | protobuf schemas (config, run, descriptor, datagen, common, runtime) |
| `workloads/` | embedded workloads: simple, tpcb, tpcc, tpch, tpcds, execute_sql |
| `docs/datagen-framework.md` | authoritative relational datagen guide |
| `docs/parallelism.md` | InsertSpec parallelism contract and tuning |

## Drivers

| Preset | Type enum | Notes |
|--------|-----------|-------|
| `pg` | DRIVER_TYPE_POSTGRES | pgxpool-based; supports plain_query, plain_bulk, native (COPY) |
| `mysql` | DRIVER_TYPE_MYSQL | sql.DB-backed via sqldriver |
| `pico` | DRIVER_TYPE_PICODATA | sql.DB-backed; `Begin()` always errors — use isolation `"none"` |
| `ydb` | DRIVER_TYPE_YDB | sql.DB-backed; native maps to BulkUpsert |
| `noop` | DRIVER_TYPE_NOOP = 5 | discards all I/O; benchmarks stroppy/framework overhead |
| *(no preset)* | DRIVER_TYPE_CSV = 6 | URL-configured CSV output driver; InsertSpec/native-only, no query path |

CSV example:
```bash
./build/stroppy run tpcb/tx -D driverType=csv \
  -D url='/tmp/tpcb-csv?merge=true&workload=tpcb' \
  --steps drop_schema,create_schema,load_data
```

Add driver: package under `pkg/driver/<name>/`, implement `driver.Driver` (`InsertSpec`, `RunQuery`, `Begin`, `Teardown`), call `RegisterDriver()` in `init()`, import in `cmd/xk6air/module.go`.

## CLI Usage

```bash
./build/stroppy run <workload> [sql-override] [flags] [-- k6-args]
```

**Positional:**
- 1st: workload — preset-relative path (`tpcc/tx`, `tpcb/procs`, `tpch/tx`), bare preset with a matching `.ts` (`simple`, `tpcds`), `.ts` file, `.sql` file, or inline SQL string
- 2nd (optional): SQL file override (e.g. `tpcc/pico`, `./workloads/tpcc/pico.sql`)

**Driver flags:**
- `-d <preset>` — driver preset: `pg`, `mysql`, `pico`, `ydb`, `noop`
- `-d '{"url":"...","bulkSize":20}'` — raw JSON driver config
- `-D key=value` — override driver field (url, driverType, defaultInsertMethod, defaultTxIsolation, errorMode, bulkSize, pool.*, postgres.*, sql.*, caCertFile, authToken, authUser, authPassword, tlsInsecureSkipVerify); multiple `-D` accumulate
- `-d1 <preset>`, `-D1 key=value` — same for second driver index (multi-driver workloads)

**Script env flags:**
- `-e KEY=VALUE` — set script ENV() value (uppercased); takes precedence over config file and script defaults

**Step control:**
- `--steps step1,step2` — run only listed steps
- `--no-steps step1` — run all steps except listed
- Mutually exclusive

**Config file:**
- Default: `stroppy-config.json` in cwd (auto-loaded if present)
- `-f prod.json` — explicit path
- Precedence (highest→lowest): real env > `-e` > config `env` > `-d/-D` > config `drivers` > script defaults

**k6 passthrough:**
- `-- <k6-args>` after separator, passed directly to k6

**Examples:**
```bash
# TPC-C with postgres
./build/stroppy run tpcc/tx -d pg -D url=postgres://... -- --vus 10 --duration 60s

# TPC-C with picodata, local SQL file (no rebuild needed)
./build/stroppy run ./workloads/tpcc/tx.ts ./workloads/tpcc/pico.sql -d pico -D url=http://...

# TPC-B
./build/stroppy run tpcb/tx -d pg -D url=postgres://... -- --duration 30s

# TPC-H

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stroppy-io/stroppy](https://github.com/stroppy-io/stroppy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
