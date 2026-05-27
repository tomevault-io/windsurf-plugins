---
trigger: always_on
description: This is a high-performance Turing machine interpreter and space-time visualizer with **three deployment targets**:
---

# Busy Beaver Blaze - AI Development Guide

## Architecture Overview

This is a high-performance Turing machine interpreter and space-time visualizer with **three deployment targets**:

1. **Python Extension (PyO3)**: High-performance frame generation for data analysis and visualization workflows
2. **Native Rust**: Maximum performance for benchmarking and standalone tools
3. **WebAssembly**: Browser-based interactive visualizations

### Core Components

- `Machine` (`src/machine.rs`): Core Turing machine interpreter with state/tape/program
- `Tape` (`src/tape.rs`): Infinite tape using two `AVec<Symbol>` (negative/nonnegative indices)
- `Spaceline` (`src/spaceline.rs`): Horizontal slice of tape at one time step, with SIMD-optimized pixel compression
- `Spacelines` (`src/spacelines.rs`): Collection of spacelines with Y-axis compression buffering
- `SpaceByTime` (`src/space_by_time.rs`): Full space-time diagram manager with adaptive sampling
- `SpaceByTimeMachine` (`src/space_by_time_machine.rs`): WebAssembly-exposed API combining machine + visualization
- `PngDataIterator` (`src/png_data_iterator.rs`): Multithreaded frame generator exposed to Python via PyO3

### Python Integration (PyO3)

The package follows the **Nine Rules for Writing Python Extensions in Rust** (see article in project docs):

1. **Single repository**: Both Rust (`src/`) and Python (`busy_beaver_blaze/`) in same repo
2. **Maturin + PyO3**: Build system in `pyproject.toml`, bindings in `src/python_bindings.rs`
3. **Translator layer**: `PyPngDataIterator` wraps native `PngDataIterator`, converts types
4. **Memory management**: Iterator yields PNG bytes to Python, avoiding shared memory complexity
5. **Error handling**: Rust `Result<T, Error>` → Python exceptions (`ValueError`, `RuntimeError`)
6. **Multithreading**: Rayon parallelism in Rust, GIL released via `py.allow_threads()`
7. **Thread control**: `part_count` parameter (defaults to CPU count) exposed to Python
8. **Type bridging**: Python strings → Rust enums, hex colors → RGB tuples
9. **Dual testing**: Rust tests (`cargo test`) + Python tests (`pytest`)

**Architecture layers**:

- **Python side** (`busy_beaver_blaze/`): Pure Python `Machine` class (notebooks), `log_step_iterator()` helper, `create_frame()` image post-processing
- **Rust translator** (`src/python_bindings.rs`): PyO3 `#[pyclass]` wrappers, type conversions, GIL management
- **Rust core** (`src/*.rs`): "Nice" Rust functions with native types, generic implementations, multithreading

**Coexistence strategy**: Pure Python and Rust implementations coexist in same namespace. Notebooks can use `Machine` (pure Python) for prototyping and `PngDataIterator` (Rust) for production runs.

**Type stub maintenance** (`busy_beaver_blaze/__init__.pyi`):

- Provides type hints for Rust classes and monkey-patched Python methods
- **Must be manually updated** when:
  - PyO3 bindings change in `src/python_bindings.rs` (add/remove/modify methods on `Visualizer`, `PngDataIterator`, etc.)
  - Monkey-patched methods change in `busy_beaver_blaze/interactive.py` (e.g., `Visualizer.run()`, `Visualizer.run_live()`)
- Run `mypy busy_beaver_blaze/` to catch type mismatches
- Run `pytest --doctest-modules` to verify examples match signatures
- Keep stub minimal: only stub Rust bindings and runtime-added methods, not pure Python with inline types

### Performance Architecture

**Adaptive Sampling**: Memory scales with image size, NOT step count or tape width

- Starts recording full tape at each step
- If tape/steps exceed 2x image size → halves sampling rate
- Uses `PowerOfTwo` types for stride calculations (`src/power_of_two.rs`)

**SIMD Optimization**: Controlled by `simd` feature flag

- Pixel binning uses SIMD lanes (8, 16, 32, 64) for averaging tape symbols
- Falls back to iterator-based implementations when SIMD unavailable
- Memory alignment via `ALIGN: usize = 64` constant

**Diff Row Optimization**: Controlled by `diff_row` feature flag (default ON)

- Optimizes frame generation by copying the previous spaceline and recomputing only the single pixel that can change per machine step.
- Gate is in `src/space_by_time.rs: snapshot()`; recompute-from-scratch fallback is used when `--no-default-features` or when `diff_row` is disabled.
- Pixel recompute is implemented in `src/spaceline.rs: redo_pixel()`.
- Build examples:
  - Default (SIMD + Diff Row): `cargo check`
  - Disable both: `cargo check --no-default-features`
  - SIMD only: `cargo check --no-default-features --features simd`
  - Diff Row only: `cargo check --no-default-features --features diff_row`

## Variable Naming Conventions

Avoid single-character variables; use descriptive names:

- ❌ `i`, `j`, `x`, `y`, `a`, `b`
- ✅ `read_index`, `write_index`, `first_pixel`, `second_pixel`

Project patterns:

- `x_goal`/`y_goal`: Target image dimensions
- `x_stride`/`y_stride`: Sampling rates (must be `PowerOfTwo`)
- `step_index`: Current machine step number
- `tape_index`: Current head position (can be negative)
- `select`: Which symbol to visualize (`NonZeroU8`)

## Comment Conventions

Use `cmk00`/`cmk0` prefix for TODO items (author's initials + priority):

```rust
// cmk00 high priority task

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarlKCarlK/busy_beaver_blaze](https://github.com/CarlKCarlK/busy_beaver_blaze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
