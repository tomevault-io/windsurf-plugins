---
trigger: always_on
description: PostGIS-compatible spatial UDF library for ClickHouse, compiled to WASM.
---

# chgeos — CLAUDE.md

PostGIS-compatible spatial UDF library for ClickHouse, compiled to WASM.
Uses GEOS 3.12+ for geometry operations. C++23.

## Build system

```bash
# Native build (tests only — can't link WASM binary with native clang)
cmake -B build -G Ninja          # first time only
ninja -C build                   # build + link chgeos_tests
ctest --output-on-failure -C build  # run all 261 tests

# WASM build (Emscripten, pre-configured in build_wasm/)
ninja -C build_wasm              # produces build_wasm/bin/chgeos.wasm
```

ClickHouse binary: `../ClickHouse/build/programs/clickhouse`
(sibling directory, built from source)

Inspect WASM exports: `wasm-tools dump build_wasm/bin/chgeos.wasm | grep <name>`

LSP shows many false-positive errors for GEOS/CH headers — ignore them. The real compiler is always the source of truth.

## Running ClickHouse server

Already running as a background process:
```
../ClickHouse/build/programs/clickhouse server \
  --config-file=clickhouse/config-test.xml \
  2>/tmp/ch-server.log &
```

After starting, wait for it to finish loading (WASM module takes ~40s):
```bash
for i in $(seq 1 60); do
  ../ClickHouse/build/programs/clickhouse client --port 19000 --query "SELECT 1" 2>/dev/null && echo "ready" && break
  sleep 1
done
```

Logs: `tail -f /tmp/ch-server.log`

Connect: `../ClickHouse/build/programs/clickhouse client --port 19000`

## Reloading chgeos.wasm after a build

Only needed when the **WASM binary changes** (`ninja -C build_wasm`). For pure ClickHouse C++ changes, just restart the CH server — no WASM reload required.

Use `scripts/reload.sh` (does all steps below automatically).

Must drop all functions first — ClickHouse won't delete a module in use:

```bash
CH="../ClickHouse/build/programs/clickhouse"
# 1. Drop all registered functions
grep -oE "^CREATE OR REPLACE FUNCTION [a-z0-9_]+" clickhouse/create.sql \
  | sed 's/CREATE OR REPLACE FUNCTION /DROP FUNCTION IF EXISTS /' \
  | sed 's/$/ ;/' \
  | $CH client --port 19000 --multiquery
# 2. Delete module
$CH client --port 19000 --query "DELETE FROM system.webassembly_modules WHERE name='chgeos'"
# 3. Copy WASM to user_files and insert
cp build_wasm/bin/chgeos.wasm tmp/data/user_files/chgeos.wasm
$CH client --port 19000 --query \
  "INSERT INTO system.webassembly_modules (name, code) VALUES ('chgeos', file('chgeos.wasm'))"
# 4. Recreate all functions
$CH client --port 19000 --multiquery < clickhouse/create.sql
```

Note: `system.webassembly_functions` does not exist. Use `system.functions WHERE origin != 'System'` to inspect registered UDFs.

## Benchmarks

SF1 data: `/Users/bacek/src/spatial-bench/sf1/` (6M trip rows)

```bash
BENCH_RUNS=5 ./scripts/bench_sf1.sh ../ClickHouse/build/programs/clickhouse \
  /Users/bacek/src/spatial-bench/sf1 Q1
```

Optional third argument filters to a single query (Q1, Q3, etc.).

For manual ad-hoc queries, run the CH client from the data directory so that
`file('trip.parquet')` resolves correctly:

```bash
CH=~/src/ClickHouse/build/programs/clickhouse
cd /Users/bacek/src/spatial-bench/sf1   # or sf10
$CH client --port 19000 --query "SELECT count() FROM file('trip.parquet', Parquet)"
```

Reference numbers after PreparedGeometry optimization (April 2026, 6M rows):
| Variant | Q1 avg | Q3 avg |
|---------|--------|--------|
| msgpack | 293ms  | 355ms  |
| col     | 113ms  | 101ms  |

## Architecture

### Three wire formats

**MsgPack** (`src/msgpack.hpp`, `src/mem.hpp`):
- One call per row; ClickHouse serializes each row as a msgpack sequence
- `impl_wrapper(buf, n, fn_impl)` — unpacks args row by row
- Registered via `CH_UDF_FUNC` macro

**RowBinary** (`src/rowbinary.hpp`):
- One call per batch; generic `rowbinary_impl_wrapper` deduces types from `_impl`
- Registered via `CH_UDF_RB_ONLY` / `CH_UDF_RB_BBOX2` macros

**COLUMNAR_V1** (`src/columnar.hpp`):
- One call for all N rows; ClickHouse sends columns (not rows)
- Constant columns (`COL_IS_CONST` flag) send one value broadcast to all rows
- `columnar_impl_wrapper(buf, n, fn_impl, ...)` — single generic template
- Registered via `CH_UDF_COL` / `CH_UDF_COL_BBOX2` / `CH_UDF_COL_PRED3` macros

### Function naming

Each function can have up to three SQL names registered in `clickhouse/create.sql`:

| Suffix | ABI | Example | Registered as |
|--------|-----|---------|---------------|
| `_mp` | MsgPack or RowBinary | `st_contains_mp` | direct WASM binding |
| `_col` | COLUMNAR_V1 | `st_contains_col` | direct WASM binding |
| *(none)* | SQL alias | `st_contains` | `AS (a, b) -> st_contains_col(a, b)` |

Canonical (unsuffixed) aliases point to `_col` when available, `_mp` otherwise.
Users call canonical names; suffixed names are available for explicit path selection.

### Source layout

```
src/
  main.cpp              — all UDF registrations (macros only, no logic)
  columnar.hpp          — COLUMNAR_V1 wire format, ColView, columnar_impl_wrapper
  rowbinary.hpp         — RowBinary wire format, rowbinary_impl_wrapper
  msgpack.hpp           — MsgPack wire format, impl_wrapper
  mem.hpp / mem.cpp     — raw_buffer, clickhouse_create_buffer, etc.
  col_prep_op.hpp       — ColPrepOp and ColPrepDistOp type aliases
  functions.hpp         — includes all function headers
  functions/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bacek/chgeos](https://github.com/bacek/chgeos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
