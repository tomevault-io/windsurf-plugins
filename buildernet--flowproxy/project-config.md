---
trigger: always_on
description: This guide helps contributors deliver changes to the project safely and predictably.
---

# Repository Guidelines

This guide helps contributors deliver changes to the project safely and predictably.

## Project Structure & Module Organization

### Entry Points & Core Structure

- The entrypoint binary lives in `src/main.rs` and delegates to the library crate in `src/lib.rs`.
- CLI arguments are defined in `src/cli.rs` using the `clap` crate.
- The `src/runner/` module handles application lifecycle and HTTP server setup.

### Domain Modules

Core domain logic is organized under `src/`:

- **`ingress/`**: HTTP handlers for three endpoints:
  - `user.rs`: Public API for bundle/transaction submission
  - `system.rs`: Internal API for system operations
  - `builder.rs`: Builder stats endpoint
  - Handles validation, entity scoring, rate limiting, and queuing
- **`forwarder.rs`**: Relays bundles to local builder via HTTP and peer proxies via BuilderHub
- **`priority/`**: Multi-level priority queue system (High/Medium/Low)
  - `mod.rs`: PriorityQueues implementation
  - `pchannel.rs`: Custom priority channels with backoff strategies
- **`entity.rs`**: Entity tracking, scoring, and spam detection logic
- **`rate_limit.rs`**: Per-entity rate limiting with sliding windows
- **`validation.rs`**: EVM transaction validation (intrinsic gas, initcode, chain ID, EIP-4844/7702)
- **`builderhub.rs`**: Peer discovery and management via BuilderHub integration
- **`indexer/`**: Data persistence layer
  - `click/`: ClickHouse indexer with async batching
  - `parq.rs`: Parquet file-based storage
- **`cache.rs`**: Order and signer caching with TTL-based LRU eviction
- **`metrics.rs`**: Prometheus metrics for observability
- **`primitives/`**: Bundle types and encoding utilities
- **`jsonrpc.rs`**: JSON-RPC protocol handling
- **`tasks/`**: Task execution and graceful shutdown coordination

### Supporting Code

- **`src/utils.rs`**: Shared helper functions
- **`benches/`**: Criterion performance benchmarks
- **`fixtures/`**: SQL schema files for ClickHouse tables
- **`simulation/`**: Load testing harness with Docker Compose
- **`tests/`**: Integration tests with reusable fixtures in `tests/common/`

### Guidelines for New Code

- Place new code in the closest domain module and expose it through `lib.rs`.
- For cross-cutting concerns, consider adding to `utils.rs` or creating a new focused module.
- Keep modules focused and cohesive; split large modules into submodules when they exceed ~500 lines.

## Build, Test, and Development Commands

### Building

- `cargo build` compiles the binary with the default dev profile.
- `cargo build --release` produces an optimized build with LTO enabled.
- `just build-reproducible` creates a verifiable production build (uses the `reproducible` profile).
- `cargo run -- --help` prints the CLI flags defined in `src/cli.rs` and is the fastest smoke-test.

### Testing

- `just test` runs the test suite using nextest (faster parallel test runner).
- `cargo test` executes unit and integration tests; add `-- --nocapture` when debugging async failures.
- `cargo bench` runs Criterion benchmarks (validation, signature verification, etc.).
- Integration tests use `testcontainers` for ClickHouse and require Docker to be running.

### Code Quality

- `just fmt` or `cargo +nightly fmt --all` applies formatting rules from `rustfmt.toml`.
- `just clippy` or `cargo clippy --all-targets --all-features -- -D warnings` enforces lints.
- All linting configuration lives in `Cargo.toml` under `[lints.clippy]`.

### Database Operations

- `just provision-db` creates the ClickHouse database and tables from `fixtures/`.
- `just reset-db` drops and recreates tables (destructive operation).
- `just extract-data <FILE>` exports data to Parquet format.

### Justfile Commands

Run `just --list` to see all available commands. The justfile automates common workflows and should be the preferred method for development tasks.

## Coding Style & Naming Conventions

- Keep Rust code within 100 columns (enforced by rustfmt).
- Use `rustfmt` for formatting; it is authoritative and will reorder imports at crate granularity.
- Use `snake_case` for modules and functions, `UpperCamelCase` for types, and reserve `SCREAMING_SNAKE_CASE` for constants.
- Prefer structured errors (`thiserror`, `eyre`) over panics, and bubble fallible calls with `?`.
- When adding public APIs, gate re-exports in `lib.rs`.
- Document public APIs with concise rustdoc comments (`///` for public items).
- Use `tracing::instrument` for function-level observability on hot paths and error scenarios.
- Prefer immutable data structures; use `Arc` for shared ownership across async tasks.
- Keep async functions small and focused; extract complex logic into sync helper functions when possible.

## Testing Guidelines

### Test Organization

- Unit tests live in the same file as the code they test, in a `#[cfg(test)]` module.
- Integration tests are in the `tests/` directory, organized by feature area.
- Shared test utilities and fixtures belong in `tests/common/`.
- Async flows rely on `#[tokio::test]`, so ensure all async test helpers are properly propagated.

### Naming Conventions

Name test functions with the behavior under test, following the pattern:

- `<module>_<scenario>_<expected_outcome>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BuilderNet/FlowProxy](https://github.com/BuilderNet/FlowProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
