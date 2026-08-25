---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A benchmark suite that measures in-memory query performance over public
[NYSE TAQ](https://ftp.nyse.com/Historical%20Data%20Samples/DAILY%20TAQ/) data across
KDB-X (q-sql), KDB-X SQL, pykx, DuckDB, chDB, Polars (eager and lazy) and Pandas. It also
contains the static dashboard ([index.html](index.html)) published as GitHub Pages at
benchmark.kx.com (see [CNAME](CNAME)).

[README.md](README.md) is the canonical user-facing documentation and stays authoritative —
when this file or [.claude/skills/run-nyse-taq-benchmarks/SKILL.md](.claude/skills/run-nyse-taq-benchmarks/SKILL.md)
disagrees with it, re-read the README. Keep all three in sync when changing behaviour.

## Commands

All scripts assume the **repository root** as CWD (they reference `./src`, `./artifacts`,
`./external` relatively; `iostat.py` shells out to `./src/resolve_device.sh`).

```bash
# One-time: the download scripts and shared bash helpers live in the taq submodule
git submodule update --init --recursive
```

### Test

There is a single end-to-end smoke test — no unit-test framework, no linter config:

```bash
./test/inmemory.sh
```

It builds a tiny kdb+ and Parquet DB from the submodule's test PSVs
(`external/kx/taq/test/data`, no download needed), runs **both** benchmark drivers with
`artifacts/parameters/test`, and regenerates a dashboard JS file. It only asserts the pipeline
runs; it checks nothing about the numbers. Run it after touching any runner, driver, query file
or parser.

### Full pipeline

```bash
export SIZE=tiny                       # tiny|small|medium|large|xlarge|full (see README Step 1)
export NYSEBENCHMARKDIR=$PWD/DATA
export DATADATE=20260401               # this changes as NYSE publishes new data

./external/kx/taq/scripts/getPSVs.sh --csvdir ${NYSEBENCHMARKDIR}/${SIZE}/psv --dates ${DATADATE} --size ${SIZE}

DATAFORMAT=kdb ./generateDB.sh ${NYSEBENCHMARKDIR}/${SIZE}/psv ${NYSEBENCHMARKDIR}/${SIZE}/kdb ${DATADATE}
SYMBOLSTOREDAS=ROWGROUP DATAFORMAT=parquet ./generateDB.sh ${NYSEBENCHMARKDIR}/${SIZE}/psv ${NYSEBENCHMARKDIR}/${SIZE}/parquet/rowgroup ${DATADATE}

./benchmarks/inmemory/queryEngines.sh --db-dir ${NYSEBENCHMARKDIR}/${SIZE} \
  --param-dir ./artifacts/parameters/${SIZE} --datadate ${DATADATE} \
  --threads "4 16" --result-dir ./results/inmemory/${SIZE}/$(date +%Y%m%d_%H%M)
```

`generateDB.sh` reads `SIZE` (mapped to a letter range by the submodule's `get_letters`),
`DATAFORMAT` and `SYMBOLSTOREDAS` from the environment; `--db-dir` is the **per-size**
directory — the drivers append `kdb` / `parquet/rowgroup` themselves.

### Narrowing a run while iterating

Full sweeps take hours; almost all development uses a subset:

```bash
./benchmarks/inmemory/queryEngines.sh --db-dir ... --param-dir ... --datadate ... \
  --threads "4" --engines kdb,duckdb --solutions "KDB-X,DuckDB (Index)" --idx 40-44
```

`--idx` accepts `42`, `32,42,50` or `40-44`. `--engines` picks engines; `--solutions` picks the
named variants within them (`"ALL"` runs every variant). Non-selected queries appear in the
output as `idxfiltered` / `tagfiltered` / `instrumentfiltered` rows rather than being dropped.

A single runner can also be invoked directly, which is the fastest debug loop (note `FLUSH`
is mandatory for both runners):

```bash
FLUSH=./flush/noflush.sh q ./src/runQueries.q -db ${NYSEBENCHMARKDIR}/${SIZE}/kdb \
  -storage_backend memory -date ${DATADATE} -paramdir ./artifacts/parameters/${SIZE} \
  -queryfile ./artifacts/queries/inmemory/kdb.psv \
  -querymeta ./artifacts/queries/inmemory/querymeta.psv -sortcols time -indexon sym -idx 42 -debug

FLUSH=./flush/noflush.sh uv run pysrc/queryrunner/main.py -db ${NYSEBENCHMARKDIR}/${SIZE}/parquet/rowgroup \
  -storage_backend memory -engine duckdb_con -date ${DATADATE} \
  -paramdir ./artifacts/parameters/${SIZE} -queryfile ./artifacts/queries/inmemory/duckdb.psv \
  -querymeta ./artifacts/queries/inmemory/querymeta.psv -sortcols time -indexon sym -idx 42
```

Both accept `-help` (q) / `--help` (Python).

### Supporting tools

```bash
# Cross-engine output equivalence (see README "Verifying Query Output Correctness")
./benchmarks/inmemory/queryEngines.sh ... --query-output-dir ./results/inmemory/output
q src/compareOutput.q -querymeta ./artifacts/queries/inmemory/querymeta.psv \
  -queryoutput1 ./results/inmemory/output/KDB-X -queryoutput2 ./results/inmemory/output/DuckDB_Index_

# Regenerate dashboard data for a size (also refreshes querymeta.generated.js)
uv run pysrc/convertToJSFormat.py ./results/inmemory/${SIZE} ./results/inmemory/${SIZE}/data.generated.js

# Query parameter files for a new SIZE
q artifacts/parameters/genParameters.q -db ${NYSEBENCHMARKDIR}/${SIZE}/kdb -dst ./artifacts/parameters/${SIZE}

# Renumber the idx column after inserting a query mid-file
./artifacts/queries/reindex.sh artifacts/queries/inmemory/*.psv
```

## Architecture

### Four-stage pipeline

`getPSVs.sh` (submodule) → parser (`src/taqToKDB.q` for kdb+, `pysrc/taqToParquet/` for
Hive-partitioned Parquet, both behind `generateDB.sh`) → benchmark driver → merged

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KxSystems/NYSETAQBenchmarks](https://github.com/KxSystems/NYSETAQBenchmarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
