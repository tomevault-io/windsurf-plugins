---
trigger: always_on
description: - This template targets Rust workspaces only.
---

# Rust Workspace Agent Instructions

## Scope

- This template targets Rust workspaces only.
- `bin/` contains CLI binary crates.
- `crates/` contains reusable library crates.
- No frontend/web-framework-specific assumptions.

## Cargo Workspace Rules (Critical)

1. Never manually type dependency versions in `Cargo.toml`; use `cargo add`.
2. Add workspace-level dependencies with:

   ```bash
   cargo add <crate> --workspace
   ```

3. Add sub-crate dependencies with:

   ```bash
   cargo add <crate> -p <crate-name> --workspace
   ```

4. Root `[workspace.dependencies]` must not carry features by default.
5. Sub-crates must use `workspace = true` for `version`, `edition`, and shared dependencies.

## Preferred Dependencies

When introducing new dependencies, prefer these crates and always use the latest stable version (via `cargo add`):

- `clap` — CLI argument parsing
- `config` — configuration management
- `eyre` — application-level error handling
- `serde` — serialization/deserialization
- `thiserror` — library error types
- `tokio` — async runtime
- `tracing` — structured logging
- `tracing-subscriber` — log subscriber
- `tracing-opentelemetry` — tracing ↔ OpenTelemetry bridge
- `opentelemetry` — metrics/traces API
- `opentelemetry-otlp` — OTLP gRPC exporter
- `sqlx` — async SQL driver
- `utoipa` — OpenAPI doc generation
- `utoipa-swagger-ui` — Swagger UI
- `arc-swap` — atomic swap for `Arc`
- `hpx` — HTTP client (this project)
- `scc` — concurrent map/set
- `winnow` — parser combinators
- `shadow-rs` — build info
- `ecdysis` — graceful restart/reload

## Dependency Priority and Forbidden Choices

- HTTP client preference: `hpx` (with `rustls`) over `reqwest`.
- Concurrent map/set preference: `scc` over `dashmap` and `RwLock<HashMap<...>>`.
- Parsing preference: `winnow` or `pest` over ad-hoc manual parsing.
- Read-heavy shared state: `arc-swap` over `RwLock`.
- Forbidden by default: `anyhow`, `log`, `reqwest`, `dashmap`.

## Engineering Principles

### Rust Implementation Guidelines

1. Error handling:
   - Application layer: `eyre`.
   - Library layer: `thiserror`.
2. Database (`sqlx`):
   - Prefer runtime queries (`sqlx::query_as`).
   - DB structs should derive `sqlx::FromRow`.
   - Avoid compile-time `sqlx::query!` macros by default.
3. Concurrency:
   - Prefer lock-free/container-first approaches (`scc`, `ArcSwap`).
   - Avoid `Arc<Mutex<T>>` when better alternatives are available.
4. Observability:
   - Logging: `tracing` only.
   - Metrics/traces: OpenTelemetry OTLP gRPC.
   - Prometheus should not be the default instrumentation path.
5. API docs:
   - Generate OpenAPI with `utoipa` when exposing HTTP APIs.
6. Configuration:
   - Use the `config` crate and external configuration files (prefer TOML).
7. Binaries:
   - Use `ecdysis` for graceful restart/reload flows in daemon/server binaries.
8. Safety:
   - Use `unsafe` only when strictly necessary and document the safety invariants.

### Key Design Principles

- Modularity: Design each crate so it can be used as a standalone library with clear boundaries and minimal hidden coupling.
- Performance: Prefer architectures that support parallelism, memory-mapped I/O for large read-heavy workloads, optimized data structures, and lock-free data types.
- Extensibility: Use traits and generic types to support multiple implementations without invasive refactors.
- Type Safety: Maintain strong static typing across interfaces and internals, with minimal use of dynamic dispatch.

### Performance Considerations

- Avoid allocations in hot paths; prefer references and borrowing to reduce allocation and copy overhead.
- Use `rayon` for CPU-bound parallel processing.
- Use `tokio` async/await for I/O-bound concurrency.

### Concurrency and Async Execution

- Prefer atomic types (`AtomicUsize`, `AtomicBool`, etc.) with explicit `Ordering` for simple shared state.
- Use `scc` for highly concurrent maps/sets; avoid `Arc<RwLock<HashMap<...>>>` and `Arc<Mutex<HashMap<...>>>` on hot paths.
- Use `moka` for concurrent caches instead of custom LRU implementations.
- Prefer `parking_lot::{Mutex, RwLock}` over `std::sync` locks for synchronous locking.
- Release `std::sync::Mutex` and `parking_lot::Mutex` guards before hitting any `.await` point.
- Use `tokio::sync::Mutex` for locks that span across `.await` points.
- Use `tokio::task::spawn_blocking` for CPU-bound work and blocking I/O.
- Batch work or use bounded worker patterns instead of spawning massive volumes of tiny Tokio tasks.
- Channel selection:
  - Async-to-Async: `tokio::sync::mpsc` / `tokio::sync::broadcast`
  - Sync/MPMC: `crossbeam-channel` or `flume`
  - Avoid `std::sync::mpsc`

### Memory and Allocation

- For binary server applications, configure `tikv-jemallocator` or `mimalloc`.
- For trusted internal hash keys, prefer `ahash` or `rustc-hash` over default SipHash-based maps.
- Use `compact_str` or `smol_str` for small-string-heavy paths.
- Prefer `beef::Cow` over `std::borrow::Cow` when minimizing footprint.
- Use `bytes::Bytes` / `bytes::BytesMut` for network buffers; pass `Bytes` instead of cloning `Vec<u8>`.
- For critical serialization hot paths, prefer `rkyv` or `zerocopy`; reserve `serde_json` for config and non-critical APIs.

### Type and Layout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longcipher/hpx](https://github.com/longcipher/hpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
