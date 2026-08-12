---
trigger: always_on
description: QuantWave is a high-performance, Polars-native technical analysis library for Rust. It extends `talib-rs-core` with modern indicators, Ehlers DSP suites, and ML feature engineering tools, ensuring bit-identical results between batch processing (Polars) and real-time streaming.
---

# QuantWave: Project Instructions

QuantWave is a high-performance, Polars-native technical analysis library for Rust. It extends `talib-rs-core` with modern indicators, Ehlers DSP suites, and ML feature engineering tools, ensuring bit-identical results between batch processing (Polars) and real-time streaming.

## 🏗 Architecture Overview

The project is structured as a Rust workspace to maximize modularity and performance:

- **`quantwave-core`**: The engine containing core traits (`Next<T>`), state machines, and streaming implementations.
- **`quantwave-polars`**: High-level Polars integration providing the `.ta()` namespace on LazyFrames and Series.
- **`quantwave-backtest`**: Vectorized/streaming backtest engine + performance metrics.
- **`quantwave-py`**: The single PyO3 (abi3) extension. One cdylib exposes the indicator classes/batch fns (`quantwave._quantwave`), the backtest bindings (`quantwave._backtest`), and the native `#[polars_expr]` plugins behind the `pl.col().ta` namespace. Built with one `maturin build` → one `cp39-abi3` wheel (no wheel-merge step).

## 🛠 Building and Running

### Prerequisites
- Rust (2024 edition)
- `cargo-expand` (recommended for macro debugging)

### Key Commands
- **Build All**: `cargo build`
- **Test All**: `cargo nextest run` (MANDATORY: Use nextest only)
- **Install pre-push gate**: `./scripts/install-git-hooks.sh` — runs verify before each push (CI runs fast doc/metadata sanity only)
- **Verify (full gate)**: `./scripts/quantwave_verify.sh` — metadata drift + nextest + pytest smoke (cached via `scripts/verify_cache.py`; `VERIFY_NO_CACHE=1` to force) (cached via `scripts/verify_cache.py`; `VERIFY_NO_CACHE=1` to force)
- **Run Benchmarks**: `cargo bench`
- **Check Linting**: `cargo clippy`
- **Check Formatting**: `./scripts/rustfmt_check.sh` — ⚠️ **never `cargo fmt --all`**. `quantwave-core/src/options_india/iv_solver.rs` holds Horner-form polynomial approximations (~12 levels of nested parens on 500-char lines) that make rustfmt spin at 100% CPU indefinitely. rustfmt follows `mod` declarations, so `src/lib.rs` and `options_india/mod.rs` inherit the hang and the whole crate becomes unformattable via cargo. This script batches and skips those paths — full workspace in ~1.2s. Interrupted `cargo fmt` runs leave **orphaned rustfmt children** burning CPU; clean up with `pkill -f "bin/rustfmt"; pkill -f cargo-fmt`.

## 🧪 Development Conventions

### 1. The "Universal Indicator" Pattern
Every indicator must implement the `Next<Input>` trait. This single source of mathematical truth powers both the streaming structs and the Polars plugins.

### 2. Parity & Validation
- **Streaming-Batch Parity**: Every indicator must have a `proptest` that asserts `Batch(data) == Streaming.collect(data)` using `approx` tolerances.
- **Gold Standard**: Reference data is stored in `quantwave-core/tests/gold_standard/*.json`. All implementations must match these industry-standard vectors.
- **Tests Location**: ALL integration tests and gold standard files MUST reside in `quantwave-core/tests/`. Root-level `tests/` folders are prohibited.
- SOURCE of calculation for all indicators must be recorded. IF you do not have a source do not assume, validate with the human before assuming the source. Research and give options for source.

### Indicator Formula References
When implementing indicators, refer to the following authoritative sources for calculation logic and edge-case handling:
- **TradingView (Pine Script):** De facto standard for retail algorithmic trading.
- **Devexperts:** https://devexperts.com/dxcharts/kb/docs/indicators
- **Sierra Chart:** https://www.sierrachart.com/index.php?page=doc/TechnicalStudiesReference.php
- **QuantConnect:** https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/wave-trend-oscillator
- **MQL5:** https://www.mql5.com/en/articles/indicators
- **StockCharts:** https://chartschool.stockcharts.com/table-of-contents/overview

### 3. Depth over Breadth
Prioritize generic, extensible components. For example, moving averages should support swappable smoothing algorithms (SMA, EMA, HMA) via the `SmoothingAlgorithm` trait.

### 4. Performance
- Use **Polars Expression Plugins** for all custom vectorized logic.
- Avoid memory copies; operate directly on Arrow buffers.
- Leverage `talib-rs-core`'s SIMD-optimized foundations for classic indicators.

## 🗺 Roadmap (Phase 1)
- [ ] Initialize workspace and foundational traits.
- [ ] Implement `SuperTrend` as the "Steel Thread" indicator.
- [ ] Establish the `gold_standard` testing infrastructure.


This project uses br (beads_rust) for issue tracking. br replaced bd
(beads/Dolt) on 2026-07-26 after the bd Dolt store's journal became
corrupted; issues were recovered from a pre-corruption JSONL export and
migrated into br's SQLite + JSONL store. bd's original data is preserved
(not deleted) in `.beads.bak-20260726-092341/` and
`.beads.bd-legacy-20260726/` at the repo root.

- Run `br robot-docs guide` for workflow context and command guidance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lavs9/quantwave](https://github.com/lavs9/quantwave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
