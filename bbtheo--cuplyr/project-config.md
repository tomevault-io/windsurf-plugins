---
trigger: always_on
description: This repo mixes R and C++ (Rcpp) for GPU-backed dplyr-like operations using libcudf. The trickiest parts are build tooling, GPU availability, and keeping R-level schemas aligned with GPU types.
---

# cuplyr Agent Notes

This repo mixes R and C++ (Rcpp) for GPU-backed dplyr-like operations using libcudf. The trickiest parts are build tooling, GPU availability, and keeping R-level schemas aligned with GPU types.

## Local Dev (pixi)
### Fast edit/run loop
1) `pixi run configure` (only when CUDA/cudf paths change).
2) Edit code.
3) `pixi run install` (full rebuild) or `pixi run load-dev` (fast reload in R).
4) Spot-check in R (e.g., `gpu_details()`, `tbl_gpu(mtcars)`).

### When to use each task
- `pixi run load-dev`: tight inner loop for R changes or quick checks.
- `pixi run install`: after C++ changes or when exports changed.
- `pixi run dev`: when you suspect stale artifacts (clean + rebuild).
- `pixi run test`: always run after implementing a feature (requires GPU).

## Core Data Structures
- `tbl_gpu` is a list with `ptr` (externalptr), `schema` (`names` + `types`), `groups`, `exec_mode` (`"eager"`/`"lazy"`), and `lazy_ops` (AST node or `NULL`).

## Type Mappings (R -> GPU)
- logical -> `BOOL8`
- integer -> `INT32`
- double -> `FLOAT64`
- character -> `STRING`
- Date -> `TIMESTAMP_DAYS`
- POSIXct -> `TIMESTAMP_MICROSECONDS`
- factor -> `INT32` (codes only)
- integer64 -> `FLOAT64` (precision loss > 2^53, warn)

## Current C++ Layout (split from transfer.cpp)
- `src/transfer_io.cpp`: R <-> GPU conversion, collect/head/df_to_gpu.
- `src/ops_filter.cpp`: filter operations and mask handling.
- `src/ops_compare.cpp`: comparison ops used for summarise temp columns.
- `src/ops_mutate.cpp`: mutate and copy/replace ops.
- `src/ops_select.cpp`: select operations.
- `src/ops_groupby.cpp`: summarise/groupby logic.
- `src/gpu_info.cpp`: device availability/info.
- `src/cuda_utils.hpp`: `check_cuda()` helper.
- `src/ops_common.hpp`: shared operator mapping helpers.
- `src/gpu_table.hpp`: pointer ownership helpers for cudf::table.

## R Entry Points
- `R/tbl-gpu.R`: `tbl_gpu()` constructor + schema metadata.
- `R/mutate.R`, `R/filter.R`, `R/select.R`, `R/arrange.R`, `R/summarise.R`: dplyr verbs.
- `R/collect.R`: pulls data back to R and warns on INT64 precision.
- `R/gpu-memory.R`: memory reporting and GC helpers.
- `R/utils.R`: shared helpers (type mapping, `wrap_gpu_call()` for clearer GPU errors).

## Known Sharp Edges (things that were hard)
- **cudf header names differ by version.** `bitmask_allocation_size_bytes` lives in `cudf/null_mask.hpp` in this environment. Avoid `cudf/bitmask.hpp`.
- **R type vs GPU type mismatch** can silently break results. Keep `gpu_type_from_r()` and `df_to_gpu()` in sync (logical/Date/POSIXct especially).
- **GPU not detected** is common in CI or local dev. Tests use `skip_if_no_gpu()`; don’t remove it.
- **Rcpp exports need regeneration** after moving/adding functions: run `Rcpp::compileAttributes()` or `devtools::document()`.
- **INT64 precision**: `gpu_collect()` returns doubles; warn when values exceed 2^53.
- **Memory growth**: each GPU op tends to allocate new tables. Replacement mutate paths are optimized, but GC still matters.
- **Join memory warnings**: joins estimate output size and warn when close to available GPU memory; actual allocation can still fail.
- **String columns**: Arrow-style storage (offsets + char data). When slicing, offsets and chars must be kept in
- **Join ordering**: cuDF join outputs are unordered; we stable-sort join maps by left_map (then right_map) in `src/ops_join.cpp` to match dplyr.
- **Join unmatched rows**: cuDF uses `JoinNoMatch` sentinel; gather treats negatives as wraparound. Current fix sanitizes join maps on CPU (replace with `nrows`) before gather and uses `out_of_bounds_policy::NULLIFY`.
- **cuDF gather API**: this environment’s `cudf::gather` signature has no `negative_index_policy`.
- **Join headers**: join APIs live under `cudf/join/join.hpp` in this env (not `cudf/join.hpp`).

## Debugging Local Build Failures
- If a cudf header can’t be found, check `pixi` environment paths and `src/Makevars`.
- Run `pixi run configure` after updating CUDA/cudf libs.
- Use `rg --files -g '*bitmask*' $CONDA_PREFIX/include/cudf` to locate moved headers.
- Join build errors: confirm `#include <cudf/join/join.hpp>` and avoid device-side Thrust unless compiling with nvcc.

## Adding New GPU Ops (local loop)
1) Implement in appropriate `src/ops_*.cpp` file.
2) Add an `// [[Rcpp::export]]` function.
3) Run `Rcpp::compileAttributes()` to regenerate `R/RcppExports.R` + `src/RcppExports.cpp`.
4) Add R wrapper in `R/*.R` and tests in `tests/testthat/`.

## When You Touch Types (local loop)
- Update schema: `R/utils.R` (`gpu_type_from_r`).
- Update collect/head conversion for new cudf types.
- Add tests for round-trip behavior.

## Joins (AST + optimizer notes)
- Lazy joins build an `ast_join` with two inputs; source pointers are attached via `set_ast_source_ptr()` before lowering.
- `infer_schema.ast_join` uses `build_join_schema()` in `R/join.R`.
- Filter pushdown across joins:
  - Inner: left-only -> left, right-only -> right.
  - Left: only left-only pushed.
  - Right: only right-only pushed.
  - Full: no side-only pushdown.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbtheo/cuplyr](https://github.com/bbtheo/cuplyr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
