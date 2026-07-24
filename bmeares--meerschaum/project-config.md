---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repo.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repo.

## Commands

### Development CLI

`./scripts/mrsm.sh <args>` runs `python -m meerschaum` with `MRSM_ROOT_DIR=./test_root` and `MRSM_PLUGINS_DIR=./tests/plugins` — never touches real config.

### Running Tests

```bash
# Run all tests (requires Docker for databases)
./scripts/test.sh db "" <pytest-args>

# Run a single test file (no Docker needed for sqlite-only tests)
MRSM_ROOT_DIR=./test_root MRSM_PLUGINS_DIR=./tests/plugins python -m pytest tests/test_pipes.py -v

# Run a specific test
MRSM_ROOT_DIR=./test_root MRSM_PLUGINS_DIR=./tests/plugins python -m pytest tests/test_pipes.py::test_sync -v

# Limit to specific DB flavors (skips others)
MRSM_TEST_FLAVORS=sqlite python -m pytest tests/ -v
```

Test connectors in `tests/connectors.py`. Default flavor set: `api,timescaledb`; set `MRSM_TEST_FLAVORS=sqlite` for fast local runs without Docker.

**Test-harness gotchas (learned the hard way):**
- **⚠️ Plain `python -m pytest` imports the STALE `~/.local` install, not the repo.** There is a separate `meerschaum` installed at `~/.local/lib/pythonX.Y/site-packages/meerschaum` (e.g. v3.3.1) that shadows the working tree under a bare `python -m pytest`. It silently lacks new modules (`connectors/sql/_partition.py`) and has older behavior, so tests fail/pass for reasons that have nothing to do with your changes. **Always force the repo onto the path: prefix `PYTHONPATH=$(pwd)`** (or just use `scripts/test.sh`, which sets things up correctly). Verify with a throwaway test asserting `meerschaum.__file__` / `meerschaum.__version__`. This single confound can burn hours — check it FIRST when a test result seems impossible.
- **Flavor keys ≠ SQL flavor names.** `tests/connectors.py` defines `timescaledb`, `timescaledb_ha`, `postgis`, `mariadb`, `mysql`, `mssql`, `oracle`, `sqlite`, `api`, `valkey`. There is **no `postgresql` key** — the PostgreSQL-family test connector is `postgis` (and `timescaledb`). `MRSM_TEST_FLAVORS=postgresql` resolves to an empty connector set and `test.sh` exits before pytest runs.
- **`scripts/test.sh` hardcodes `--ff` and `-n=auto`** (line ~134-135), so it requires `pytest-cache` and `pytest-xdist`. Passing `-p no:cacheprovider` to it breaks `--ff` (`unrecognized arguments: --ff`). To run a single test with non-default pytest flags, invoke `PYTHONPATH=$(pwd) python -m pytest` directly with the `MRSM_*` env vars exported (don't go through `test.sh`).
- **`test.sh` runs one flavor at a time** and shares `test_root` + the test-API port 8989 — never run two `test.sh` invocations (or a `test.sh` + a direct API-flavor pytest) concurrently; they collide on the API server and DB tables (the symptom is a flood of `Connection refused` / pool-exhaustion failures that look like real bugs but are pure contention).
### Fixed in this cycle: `get_sync_time(params=...)` on partitioned MariaDB

The v3.4.0 `hypertable=True` default auto-partitions datetime pipes. On **MariaDB** specifically, `SQLConnector.get_sync_time` returned `None` for a populated, partitioned pipe when `params` were passed (broke `test_parametrized_sync_time[mariadb]` and, via a `None` min bound, `test_get_data_iterator[mariadb]`). Root cause: a MariaDB 12.x optimizer bug where the CTE-wrapped `... ORDER BY <dt> DESC LIMIT 1` over a `RANGE COLUMNS`-partitioned table with a `WHERE` clause does a partition index scan that stops early and returns zero rows. (The data and DB are fine — raw `WHERE id='a'` returns the row; only the ordered-LIMIT-1 form on the partitioned table fails. MySQL/PostgreSQL/MSSQL/TimescaleDB do **not** exhibit it — MariaDB-only.) **Fix** (`meerschaum/connectors/sql/_pipes.py`, in `get_sync_time` just before `wrap_query_with_cte`): for `self.flavor == 'mariadb' and not remote and self._should_partition(pipe)`, compute the bound with `MAX(dt)` / `MIN(dt)` (which scans pruned partitions correctly) instead of `ORDER BY ... LIMIT 1`. Flavor-gated, so every other flavor and non-partitioned MariaDB keep the original path.

**Test-running lesson:** `get_sync_time` drives every incremental sync — verify changes to it with the full sqlite `test_sync.py`+`test_verify.py`+`test_pipe_data.py` AND per-flavor partition runs, not just the failing tests. (And read pytest summaries carefully: "N passed, 2 warnings" is not "2 failed".)

### Docs

```bash
./scripts/zensical.sh   # serve the user-facing docs at docs/zensical/
./scripts/pdoc.sh       # build the Python API reference docs
```

### Build

```bash
./scripts/build.sh      # cleans, builds docs, generates requirements, runs python -m build
```

## Architecture Overview

ETL framework centered on **pipes** — named data streams synced into tables. Each pipe: three keys, metadata managed by **instance connector**, data from **source connector**.

### Core Abstractions

**`SuccessTuple`** (`Tuple[bool, str]`) — universal return type for all actions, pipe methods, connector methods. Return `(True, "Success")` or `(False, "reason")`. Never raise from action-level code — catch and return `SuccessTuple`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bmeares/Meerschaum](https://github.com/bmeares/Meerschaum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
