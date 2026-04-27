---
trigger: always_on
description: Harmonize the *interface*, not the data. Surveys differ in structure; the library provides a uniform API without discarding survey-specific detail.
---

# LSMS Library

## Design Philosophy
Harmonize the *interface*, not the data. Surveys differ in structure; the library provides a uniform API without discarding survey-specific detail.

## Core API
```python
import lsms_library as ll
uga = ll.Country('Uganda')
uga.waves          # ['2005-06', '2009-10', ...]
uga.data_scheme    # ['people_last7days', 'food_acquired', ...]
food = uga.food_expenditures()  # Standardized DataFrame
```

## Country Symlinks
Root-level symlinks (e.g., `Uganda → lsms_library/countries/Uganda`) are for convenience. Actual config is under `lsms_library/countries/`.

## DVC Repository Root
**The DVC repository is rooted at `lsms_library/countries/`, NOT the top-level repo.** DVC config, remotes, and credentials all live under `lsms_library/countries/.dvc/`. All `dvc` CLI commands (pull, push, status) must be run from `lsms_library/countries/` or they will fail with missing-remote/credential errors. The `get_dataframe()` fallback chain handles this automatically when scripts run from their normal `{Country}/{wave}/_/` working directory.

## Cache Behavior (v0.7.0+)

**Writes** are redirected to `data_root()` by `_resolve_data_path` in `local_tools.py`. Default location `~/.local/share/lsms_library/{Country}/var/{table}.parquet`; override with `data_dir` in `~/.config/lsms_library/config.yml` or `LSMS_DATA_DIR` env var (env var wins).

**Reads** as of v0.7.0: `load_dataframe_with_dvc` in `country.py` does a **best-effort cache read at the top of the function**, before consulting DVC at all. If `cache_path.exists()` and `LSMS_NO_CACHE` is not set, it returns the cached parquet directly. This applies uniformly to all 40 countries: the `if not stage_infos:` branch, the stage-layer branch, and the DvcException fallback all populate the same cache that the top-of-function read picks up next time.

Empirical numbers on Savio/Lustre with `.dta` files local (see `bench/results/`):

| Phase | Pre-v0.7.0 (write-only bug) | v0.7.0+ |
|---|---|---|
| Niger `household_roster` cold rebuild | ~16s | ~14s |
| Niger second call same process | ~7s | **~0.5s** |
| Niger second call fresh subprocess | ~16s (no speedup) | **~0.9s** |
| Uganda `household_roster` second call fresh subprocess | ~12s (no speedup) | **~1.2s** |

The 10–17× cross-process speedup is the headline win. Python import of `lsms_library` (~22s on Lustre, ~30s on first cold filesystem) is now the dominant cost of any fresh-session call.

**No staleness check.** v0.7.0 deliberately ships *without* automatic invalidation. Editing a wave's `data_info.yml`, a `_/{table}.py` script, or an upstream `.dta` file does *not* trigger a rebuild on the next call. Contributors must:

- set `LSMS_NO_CACHE=1` in the environment to bypass the top read for that session, OR
- run `lsms-library cache clear --country {Country}` to remove the parquet, OR
- pass `trust_cache=False` (the default; this still hits the v0.7.0 top read — see below).

**`trust_cache=True`** is a separate, narrower short-circuit at the top of `_aggregate_wave_data` that reads the parquet and then **still calls `_finalize_result`** (so kinship expansion, spelling normalization, and the `_join_v_from_sample` augmentation all apply on the way out). It bypasses the DVC stage check, the in-process ancillary caches, and the entire `load_dataframe_with_dvc` flow including the v0.7.0 top read. Effectively a stricter version of the v0.7.0 behavior for users who *know* the cache is fresh and don't want any cache existence checks beyond the parquet itself.

**`LSMS_BUILD_BACKEND=make`** dispatches `_aggregate_wave_data` directly to `load_from_waves` (`country.py:1830`), bypassing both the v0.7.0 top read and the DVC stage layer. **It does not write to or read from the data_root cache at all.** Every call rebuilds from source. The only writes that happen under `LSMS_BUILD_BACKEND=make` are via the CLI's explicit `--out` path when invoked from a stage's `cmd:` (which is itself currently broken on most Savio nodes — see "Stage layer python3 mismatch" below).

**Stage layer python3 mismatch.** The `dvc.yaml` files for the 7 stage-layer countries (Uganda, Senegal, Malawi, Togo, Kazakhstan, Serbia, GhanaLSS) declare a `cmd:` like `cd _ && LSMS_BUILD_BACKEND=make python3 -m lsms_library.cli materialize ...`. The bare `python3` resolves to system Python, which on Savio is 3.6.8 — too old for `from importlib.metadata import ...` in `lsms_library/__init__.py`. Every `stage.reproduce()` therefore fails with `ModuleNotFoundError`, and `load_dataframe_with_dvc` falls into its outer `except (DvcException, FileNotFoundError)` handler. With the v0.7.0 fix, that handler now writes the `load_from_waves` result to the cache, so the failure is benign for warm-cache reads but still costs a wasted subprocess on cold rebuilds. Fix is to substitute `${PYTHON:-python3}` or `sys.executable` in the dvc.yaml templates; deferred because v0.8.0 retires the stage layer entirely.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ligon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
