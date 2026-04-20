---
trigger: always_on
description: This file is read automatically by supported coding agents at the start of
---

# AI Agent Rules

This file is read automatically by supported coding agents at the start of
each session. It contains the non-negotiable rules for this repository.

## Project Identity

- **Name:** CorridorKey Runtime
- **Language:** C++20 (no modules)
- **Build:** CMake 3.28+ with vcpkg manifest mode and CMake Presets
- **License:** CC BY-NC-SA 4.0

## Structural Rules (enforced — see docs/ARCHITECTURE.md for details)

- Public headers go in `include/corridorkey/` — this is the external API
- Use **PIMPL Pattern** for main classes such as `Engine` to ensure ABI stability
- Use **Symbol Visibility (hidden by default)** — only export via `CORRIDORKEY_API`
- **Zero-Copy Performance:** use `std::span` via `Image` for processing and
  `ImageBuffer` for ownership. Do not use `std::vector<float>` for large pixel data
- **SIMD Alignment:** ensure 64-byte alignment for all image allocations
- Implementation goes in `src/` subdirectories by domain:
  - `src/app/` — application orchestration, runtime contracts, diagnostics, OFX service
  - `src/cli/` — CLI only (main + arg parsing, no business logic)
  - `src/common/` — shared internal utilities (STL-only, no external deps)
  - `src/core/` — inference engine, device detection, ONNX Runtime wrapper
  - `src/frame_io/` — EXR, PNG, video I/O
  - `src/gui/` — Tauri desktop UI and bridge code
  - `src/plugins/ofx/` — OFX plugin integration over the app-layer service
  - `src/post_process/` — pure pixel math (color utils, despill, despeckle)
- Tests go in `tests/unit/`, `tests/integration/`, `tests/e2e/`, or `tests/regression/`
- External library types (`OrtSession`, `Imf::*`, `AVFrame`, etc.) never appear in
  public headers — they are wrapped in `src/`
- Do not create new top-level directories or new `src/` subdirectories without
  updating docs/ARCHITECTURE.md

## Code Standards (see docs/GUIDELINES.md for details)

- `snake_case` for functions, variables, files
- `PascalCase` for types (classes, structs, enums)
- `m_` prefix for private members
- `UPPER_SNAKE_CASE` for constants
- `.hpp` for headers, `.cpp` for implementation — no `.h` files
- Const correctness everywhere
- RAII for all resources — no raw `new`/`delete`
- Error handling via `std::expected` or `std::optional`, not exceptions for
  expected failures
- Max cognitive complexity per function: 15
- Max ~200 lines per `.cpp`, ~100 lines per `.hpp` (guideline, not hard rule)
- Prefer early returns over nested if/else
- No abbreviations in names except standard domain terms: `rgb`, `exr`, `fps`, `io`

## Testing

- Framework: Catch2 v3
- Tags: `[unit]`, `[integration]`, `[e2e]`, `[regression]`
- Bug fixes must include a regression test
- Unit tests: no I/O, no GPU, < 1 second each
- Test file naming: `test_<module>.cpp`

## Build & Infrastructure

- Use `CMakePresets.json` as the source of truth for build configurations
- Always use **vcpkg Manifest Mode** with `vcpkg-configuration.json` for baseline pinning
- Strict warnings: `-Wall -Wextra -Wpedantic -Werror` or the MSVC equivalent
- Enable **AddressSanitizer (ASAN)** in Debug presets
- Keep build behavior target-based in CMake. Do not use global include, link,
  or warning configuration when a target-scoped setting is available

## Operational Rules

- Use feature branches and PRs. Do not work directly on `main`
- `scripts/windows.ps1` is the only Windows entrypoint. Every Windows
  build, package, certification, and release runs through it via
  `-Task build | prepare-rtx | certify-rtx-artifacts | package-ofx |
  package-runtime | release | regen-rtx-release | sync-version`. Never
  call `scripts/build.ps1`, `scripts/prepare_windows_rtx_release.ps1`,
  `scripts/release_pipeline_windows.ps1`, or any other sub-script
  directly; they are internal delegates and skip the version metadata
  sync, track resolution, and validation the wrapper applies.
- When any prerequisite is missing (TensorRT-RTX SDK at
  `vendor/TensorRT-RTX/`, `vcpkg` at `VCPKG_ROOT`, `uv`, CUDA Toolkit
  12.8), fix the prerequisite. Do not route around the canonical
  pipeline.
- The only supported repo-local Windows runtime roots are
  `vendor/onnxruntime-windows-rtx` and `vendor/onnxruntime-windows-dml`
- Do not use `vendor/onnxruntime-universal` or a globally installed ONNX Runtime
  as a Windows build or release dependency
- Do not create git worktrees that shadow `vendor/`. `git worktree
  remove --force` has followed Windows junctions into `vendor/` and
  erased the real curated runtimes. If a second working copy is needed,
  stage its own curated runtimes instead of sharing them.
- Changes to the render hot path (`src/plugins/ofx/`,
  `src/core/inference_session.cpp`, `src/core/engine.cpp`,
  `src/core/gpu_prep.cpp`, `src/core/gpu_resize.cpp`,
  `src/post_process/`) must be measured against the
  `phase_8_gpu_prepare` baseline in `docs/OPTIMIZATION_MEASUREMENTS.md`
  via `scripts/run_corpus.sh` + `scripts/compare_benchmarks.py`. Reject
  the change when `avg_latency_ms` or `ort_run` regresses by more than
  10%.
- Windows packaging may proceed with a partial model set, but missing packaged
  models must be surfaced in generated inventory and validation reports.
  Missing models are reportable packaging state; invalid packaged models that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexandremendoncaalvaro/CorridorKey-Runtime](https://github.com/alexandremendoncaalvaro/CorridorKey-Runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
