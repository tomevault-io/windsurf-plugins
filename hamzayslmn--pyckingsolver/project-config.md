---
trigger: always_on
description: Python wrapper for [fontanf/packingsolver](https://github.com/fontanf/packingsolver) irregular (2D nesting) module.
---

# pyckingsolver — Agent Knowledge

Python wrapper for [fontanf/packingsolver](https://github.com/fontanf/packingsolver) irregular (2D nesting) module.  
C++ submodule pinned at `extern/packingsolver` (commit `1ad3c94e9` — 2026-07-21).
Python wrapper version: `0.6.7` (see `## v0.2.0 Breaking Changes` below).

---

## MARK: Version Tags

When bumping the Python wrapper version, update all current-version tags:

- `python/pyproject.toml` `[project].version`
- `python/pyckingsolver/__init__.py` `__version__`
- `python/uv.lock` and `test/uv.lock` — `[[package]] name = "pyckingsolver"` version
- This file's top `Python wrapper version` line
- `README.md` release/current binary note if the bundled C++ solver pin changed
- `python/pyckingsolver/types.py` top commit note if the mirrored upstream C++ commit changed
- Git release tag uses `vX.Y.Z` format, for example `v0.3.3`

Historical headings such as `v0.2.0 Breaking Changes` are not current-version tags and should not be rewritten during a release bump.

---

## MARK: Recent Upstream Changes (2026-07-05 → 2026-07-11)

Pulled `750c7d7fd` → `59f50fed3` (15 commits). Bundled binary rebuilt + re-bundled.

Only 3 commits touch `src/irregular` (the module the wrapper ships); the rest are
rectangleguillotine / box / boxstacks / onedimensional or tooling/tests, none
reachable from `packingsolver_irregular`.

| Commit | Change | Impact |
|---|---|---|
| `a13c8bf1f` | **Fix NotAnytimeDeterministic racing across algorithms in optimize()** | **Behavioral — the reason to rebuild.** Reworks how the parallel algorithms are launched/collected in every domain's `optimize()` (irregular +167 lines) so `NotAnytimeDeterministic` no longer races between threads. No API/CLI/JSON change; deterministic solves are now actually reproducible. |
| `3282a7bcf` | rectangle, irregular: print leftover mode in `Instance::format` for BinPackingWithLeftovers | stdout only — wrapper never parses `Instance::format` output. No impact. |
| `76ed100b3` | Remove unused includes across all domains | Build only. |
| `59f50fed3` / `beac97016` / `65ab2aee5` / `61b0ee0b6` / `8d379c045` / `ea670b2f8` | rectangleguillotine: cutting-cost fixes, new `BinPackingCuttingCost` objective, DP→tree-search-hypergraph rename | Other domain — not reachable from `packingsolver_irregular`. |
| `48aec0c28` / `710fd4666` / `8ad89572b` | boxstacks / box / onedimensional internal fixes + refactors | Other domain. |
| `b683f0c01` / `a2f598b07` | scripts: visualizer `--scale`, fonts, subplot aspect ratio | Tooling only. |
| `67aad9299` | Fix `sequential_strips_onedimensional` subproblem tests | Test only. |

**Wrapper impact**: zero. No new irregular CLI flags, input/output JSON
unchanged, no Python source edits required. Rebuild + re-bundle
`packingsolver_irregular.exe` to pick up the `NotAnytimeDeterministic`
determinism fix (done for this bump).

---

## MARK: Recent Upstream Changes (2026-06-30 → 2026-07-05)

Pulled `8ea3129e6` → `750c7d7fd` (22 commits). Bundled binary rebuilt + re-bundled.

| Commit | Change | Impact |
|---|---|---|
| `4c40e57e0` | **irregular: skip bins that can't fit any item in the tree search** | **Behavioral — the reason to rebuild.** Previously a bin position where nothing fit dead-ended the branch (no insertions possible). Now the next bin position is tried; skipped bins are still added to the solution **as empty bins** to keep bin positions and cost accounting consistent. Wrapper parsing already tolerates `items: []`; note that `Solution.bins` may now contain empty bins and `total_bins_used()` counts them (matches C++ `BinCost`). |
| `91f2dacf9` + 12 more | rectangleguillotine: new `sequential_strips_onedimensional` algorithm, new `maximum_number_1_cuts` / `maximum_distance_2_cuts` instance params, many `column_generation_strips` fixes | Other domain — not reachable from `packingsolver_irregular`. |
| `bbe384e4c` | `Output` gains `open_dimension_x_bound` / `open_dimension_y_bound` (common.hpp) | Only set/serialized by rectangleguillotine; irregular `--output` metrics JSON unchanged. |
| `416f2cea6` | Fix dash-underline width for X/Y column headers | Cosmetic stdout only (touches irregular `algorithm_formatter`); wrapper never parses stdout tables. |
| `c67b89360` | extern: bump `columngenerationsolver` GIT_TAG | Build only — CG algorithm internals; no API change. |
| `a519caaa5` / `43fda0773` / `e65bb1356` / `9a52f867e` / `4c40e57e0`-tests | Test restructuring into `tree_search` dirs | Test only. |

**Wrapper impact**: zero CLI/JSON sync — no new irregular flags, input/output
JSON unchanged. One semantic note: solutions may now contain empty bins (see
above). Rebuild + re-bundle `packingsolver_irregular.exe` (done for this bump).

---

## MARK: Recent Upstream Changes (2026-06-27 → 2026-06-30)

Pulled `c2c1f1f42` → `8ea3129e6` (7 commits). Bundled binary rebuilt + re-bundled.

| Commit | Change | Impact |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HamzaYslmn/pyckingsolver](https://github.com/HamzaYslmn/pyckingsolver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
