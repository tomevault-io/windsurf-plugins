---
trigger: always_on
description: You are an expert machine learning engineer and Rust systems programmer building a **local-first AutoML platform**. Everything runs on the user's machine — no cloud dependencies, no external API calls, no telemetry. Privacy and performance on commodity hardware are first-class concerns.
---

# Copilot Instructions for Kolosal AutoML

You are an expert machine learning engineer and Rust systems programmer building a **local-first AutoML platform**. Everything runs on the user's machine — no cloud dependencies, no external API calls, no telemetry. Privacy and performance on commodity hardware are first-class concerns.

When working on this codebase:
- Reason about ML correctness: numerical stability, bias-variance tradeoffs, proper train/test separation, metric selection for task type.
- Reason about systems performance: memory layout, cache locality, SIMD opportunities, zero-copy data paths, thread contention.
- Prefer algorithms that work well on single-machine scale (thousands to low millions of rows). Don't over-engineer for distributed scenarios.
- All data stays local. Models are stored in `./models/`, datasets in `./data/`. The web UI is served from local static files, not a CDN.
- When adding new ML models, follow the existing `fit()`/`predict()` pattern on `Array2<f64>`, wrap in the `TrainedModel` enum, and ensure serde support for local model persistence.
- When extending the web platform, keep the htmx + Alpine.js approach — no heavy JS frameworks, no build step for frontend assets.

## Build & Run

```bash
cargo build --release          # Build all (CLI + server)
cargo test --workspace         # Run all tests
cargo test test_name           # Run a single test by name
cargo test --test test_training # Run a specific integration test file
cargo bench --bench preprocessing  # Run preprocessing benchmarks
cargo bench --bench training       # Run training benchmarks
```

Binaries: `target/release/kolosal` (CLI) and `target/release/kolosal-server` (web server).

Start the dev server: `cargo run --package kolosal-automl --bin kolosal -- serve --port 8080`

Logging is controlled via `RUST_LOG` env var (defaults to `kolosal=info`). TTY gets human-readable output; non-TTY gets structured JSON.

## Architecture

This is a **single-crate Rust workspace** (`kolosal-automl`) that produces two binaries from one library:

- **`src/lib.rs`** — The core library. All ML functionality lives here, organized into ~25 modules. A `prelude` module re-exports the most-used types.
- **`src/main.rs`** — Entry point that dispatches CLI commands via `clap`. Running with no subcommand launches interactive mode.
- **`src/server/`** — Axum-based HTTP server with REST API. State is managed via `AppState` (Arc-shared). Routes defined in `api.rs`, handlers in `handlers.rs`.
- **`src/cli/`** — CLI command implementations (`cmd_train`, `cmd_predict`, `cmd_serve`, etc.).
- **`kolosal-web/static/`** — Frontend assets (htmx + Alpine.js) served by the Axum server.

### ML Pipeline Flow

1. **Preprocessing** (`src/preprocessing/`) — Scalers, encoders, imputers, outlier detection, feature selection. Composed via `pipeline.rs`.
2. **Training** (`src/training/`) — `TrainEngine` orchestrates model training. Models implement a common pattern with `fit()` and `predict()` methods on `Array2<f64>` inputs. `TrainedModel` enum wraps all model variants.
3. **Inference** (`src/inference/`) — Optimized prediction engine for production use.
4. **Optimizer** (`src/optimizer/`) — `HyperOptX` hyperparameter optimization with TPE/Random/Grid samplers, Bayesian optimization via Gaussian Process, and pruners (Median, Percentile, Hyperband).

### Supporting Modules

- `autopipeline` — Automatic pipeline construction based on data characteristics.
- `ensemble` — Voting and stacking ensemble methods.
- `explainability` — SHAP-like feature importance and partial dependence plots.
- `anomaly` — Isolation Forest and Local Outlier Factor.
- `drift` — Data and concept drift detection.
- `export` — Model serialization to ONNX and PMML formats.
### Serving & Production Infrastructure

The platform is designed to run as a **local production server** on a single machine. Understand these layers when working on serving or optimization:

**Server layer** (`src/server/`): Axum router with `AppState` holding all mutable state behind `tokio::sync::RwLock`. Routes are nested under `/api` (data, train, predict, system, batch, quantize, device, monitoring). Static files served via `tower_http::ServeDir`. Middleware stack: gzip compression → CORS → tracing.

**Inference engine** (`src/inference/`): `InferenceEngine` wraps a trained model + optional preprocessor behind `Arc` for shared-nothing concurrent prediction. Supports loading from disk via `InferenceEngine::load()`. Use this for serving predictions, not `TrainEngine` directly.

**Batch processing** (`src/batch/`): Two complementary systems:
- `BatchProcessor` — worker-thread pool with priority queues, adaptive batch sizing, health monitoring, and memory-aware throttling. Config-driven via `BatchProcessorConfig`.
- `DynamicBatcher` — low-latency request batcher with configurable max wait time (default 10ms) and adaptive sizing. Optimizes throughput vs. latency tradeoff.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KolosalAI/kolosal_automl](https://github.com/KolosalAI/kolosal_automl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
