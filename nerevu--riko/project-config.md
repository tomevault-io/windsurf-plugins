---
trigger: always_on
description: Python stream processing engine modeled after Yahoo! Pipes.
---

# riko

Python stream processing engine modeled after Yahoo! Pipes.

## Key Paths

| Path | Role |
|---|---|
| `riko/collections.py` | `SyncPipe`, `AsyncPipe`, `SyncCollection`, `AsyncCollection` |
| `riko/modules/` | individual pipe implementations (`fetch`, `filter`, `hash`, etc.) |
| `riko/modules/__init__.py` | re-exports the module-dev surface (`processor`/`operator`/`splitter`/`Module`, all defined in `modules/_decorators.py`); derived module catalog (`list_modules`) |
| `riko/modules/_decorators.py` | `Module` base + `processor`/`operator`/`splitter` decorators; sync/async wrappers (incl. implicit-looping auto-map over iterator sources) |
| `riko/modules/_loop.py` | loop execution — `_run_loop_sync`/`_run_loop_async`, `loop_embed_sync`/`loop_embed_async`, per-parent `_fold_parent`/`_take`/`_take_first` (see `_docs/PHASE_CHECKLISTS.md` § P7 → "Loop restructure & implicit looping") |
| `riko/parsers.py` | sync XML/HTML parsing (`xml2etree`, `LinkParser`, etc.) |
| `riko/bado/__init__.py` | async backend detection (AnyIO or empty fallback) |
| `riko/bado/io.py` | async file/URL I/O (`async_url_read`, `async_url_open`) |
| `riko/bado/itertools.py` | async itertools: `async_map` + bounded/streaming variants `async_map_stream`/`async_map_ordered_stream`, `async_merge` (bounded arrival-order feed merge), `coop_reduce`/`async_reduce`, `async_iter` |
| `riko/bado/_util.py` | async utilities (`async_sleep`, `defer_to_process`) |
| `riko/_pubsub/` | pub/sub package (`send`/`receive`/`coroutine`, `reset_pubsub`) — state via `contextvars`; `_sync.py`/`_async.py`. (`riko/utils.py`/`helpers.py` are gone — decomposed into `_io`/`_iterutils`/`_serialize`/`_strutils`/`_logging`, graph→`compile.py`, `parse_context`→`context.py`, pub/sub→`_pubsub`.) |
| `riko/exceptions.py` | `UnsupportedModuleError` (unresolved leaf module) / `UnsupportedPipelineError` (unresolved `pipe_*` sub-pipeline) — both raised in `resolve_module` in `compile.py` |
| `riko/dotdict.py` | `DotDict` — case-insensitive nested dict for pipe items (dotted keys = nested paths; see `_docs/gameplans/dotdict-parsing.md` for the data-derived-key footgun) |
| `riko/cli/compile.py` | `compile-pipe` script (`[project.scripts]` name; entry `riko.cli.compile:run`) — JSON pipeline → generated Python module (wraps `compile.compile`) |
| `riko/cli/convert_dag.py` | `convert-dag` script — bare-bones DAG → full JSON pipeline (`convert-dag`) |
| `riko/cli/gen_config.py` | `gen-config` script — regenerates `riko/types/configs.py` from the nonraw `<Name>Conf` TypedDicts in `riko/types/modules.py` (+`ruff format`) |
| `riko/types/configs.py` | generated per-module `<Name>Objconf(DynamicConf)` parse-time config types (edit `modules.py` contracts, run `gen-config` — never hand-edit) |
| `riko/transform.py` | column transformation helpers (shelved; ideas now incorporated into the `_docs/gameplans/`) |
| `docs/DAG_FORMAT.rst` | bare-bones DAG format + `convert-dag`/`compile-pipe` commands |
| `docs/MIGRATION.rst` | **consolidated** user migration guide, two parts: Part 1 = **verified** `legacy` branch → current diffs (examples run on both branches, `# LEGACY`/`# CURRENT`); Part 2 = milestone notes expanded from `CHANGES.rst`. `legacy` branch = ancestor of HEAD (last commit before legacy-removal), NOT a pre-refactor snapshot — both are v0.72.0 (AnyIO, three-tier API, ExecutionMode). Real legacy→current diffs: `Context` describe kwargs ignored, `Objconf` removed, legacy JSON loop/output forms removed, `get_path` into `__all__`. **No Twisted anywhere; `bado` is AnyIO and NOT deprecated.** |
| `docs/CHANGES.rst` | changelog; git tags = milestones (2026 refinement work = `v0.67.0`–`v0.72.0`) |
| `README.rst` + `docs/{FAQ,COOKBOOK,INSTALLATION}.rst` + `CONTRIBUTING.rst` | user-facing docs. House style: wrap riko terms in ``double backticks``; **horizontal simple/grid tables only, never `.. list-table::`**; manual `Index` line with explicit `.. _Label: #github-anchor` targets (not `.. contents::`); `√`/blank in capability matrices. Every `>>>` block is a doctest — validate with `uv run --active --no-sync manage test --no-cov --where <file>` and lint RST with docutils (grid tables/anchors). README keeps both the Huginn/Flink/Spark/Storm comparison **and** a "Choosing riko" grid (Pandas/Polars/Beam/RxPY/itertools/Luigi/Prefect). Content was incorporated from the (untracked, removable) `docs/riko-docs-complete-package/` scaffold. |
| `pyproject.toml` `[project.optional-dependencies]` | extras: `perf` (fastfeedparser, ijson, lxml), `async` (anyio, httpx), `finance` (csv2ofx) |
| `_docs/ROADMAP.md` | **the map (pure index).** ~40 lines: intro + a `## Gameplans` table (the single authoritative index of every gameplan) + a pointer to the P-track. It links out; it holds no contract. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerevu/riko](https://github.com/nerevu/riko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
