---
trigger: always_on
description: - Purpose: onboarding instructions and conventions for automated agents working in this repository (`excelstream`).
---

**Agents Guide**

- Purpose: onboarding instructions and conventions for automated agents working in this repository (`excelstream`).
- Location: root of repo. Use this file as the primary source for build/test/lint commands and style rules.

**Build & Test**

- Primary build: `cargo build --all-features` (also available via `make build`). See `Cargo.toml` for optional features.
- Release build: `cargo build --release --all-features` or `make build-release`.
- Run all tests: `cargo test --all-features` or `make test` (Makefile runs `cargo test --all-features`).
- Run a single unit test (by name): `cargo test <test_name> -- --nocapture`.
  - Example (unit test in `src`): `cargo test parse_cell_value -- --nocapture`.
- Run a single integration test (integration tests live in `tests/`): `cargo test --test integration_test <test_name> -- --nocapture`.
  - Example: `cargo test --test integration_test streaming_reads -- --nocapture`.
- Run tests for a specific target or example: `cargo test --lib` or `cargo test --example <name>`.
- Run benchmarks: `cargo bench` or `make bench` (bench harness defined in `Cargo.toml`).

**Format & Lint**

- Formatting: `cargo fmt` (Makefile target `make fmt`). Check-only: `cargo fmt -- --check` or `make fmt-check`.
- Linting: `cargo clippy --all-targets --all-features -- -D warnings` or `make clippy`.
- CI helper: `make ci` runs `fmt-check`, `clippy`, and `test` (mirrors GitHub Actions flow).

**Running examples and tools**

- Build examples: `cargo build --examples --all-features` or `make examples`.
- Run an example: `cargo run --example basic_write` or `make run-example-basic_write`.
- Build docs: `cargo doc --all-features --no-deps` or `make doc`.

**Repository targets & shortcuts**

- Makefile targets: `help`, `fmt`, `fmt-check`, `clippy`, `test`, `build`, `build-release`, `bench`, `examples`, `example-<name>`, `run-example-<name>`, `ci`.
- Primary files: `Cargo.toml`, `Makefile`, `src/lib.rs`, `src/error.rs`, `tests/integration_test.rs`.

**Code Style Guidelines**

- Formatting
  - Use `rustfmt` defaults. Run `cargo fmt` before commits. See `Makefile: fmt` and `fmt-check` targets.
  - Keep line length reasonable (~100 chars) for readability in PR diffs.

- Imports & Module Organization
  - Group imports in this order: 1) std, 2) external crates, 3) internal crate (`crate::...` or `super::...`).
  - Use explicit `use` for commonly referenced items; prefer shorter local names using `as` only when necessary to avoid collisions.
  - Re-export module-level public API from `src/lib.rs` to present a small surface area to consumers.
  - Keep modules small and focused; split large files (over ~400 lines) into submodules.

- Naming Conventions
  - Types, structs, enums, traits: `PascalCase` (e.g., `ExcelWriter`, `CsvWriter`).
  - Functions, variables, module names: `snake_case` (e.g., `write_row`, `parse_csv`).
  - Constants: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_CHUNK_SIZE`).
  - Trait names: use descriptive nouns (no mandatory `Trait` suffix) — prefer `Writer`, `Reader` when appropriate.

- Types & Ownership
  - Prefer references (`&str`, `&[u8]`) in APIs when the callee does not need ownership.
  - Use `String` and owned collections when data must be stored across async boundaries or persisted inside structs.
  - Prefer slices (`&[T]`) and iterator-based APIs over indexing to reduce allocations and copying.
  - Favor small, explicit structs over tuples for clarity in public APIs.

- Error Handling
  - Use the crate's custom error type defined in `src/error.rs` and the `thiserror` crate for conversions.
  - Prefer returning `Result<T, Error>` (crate error) from public APIs.
  - Use the `?` operator to propagate errors; map errors to crate error variants near FFI/IO boundaries.
  - Avoid `unwrap()`/`expect()` in library code. Tests or examples may use `expect` for brevity but prefer `?` or `assert` for test assertions.
  - When converting external errors, preserve context with `.context()`-style messages (or `map_err(|e| Error::Io(format!("...: {}", e)))`).

- Panics & Safety
  - Library code must avoid panics on user input; return errors instead. Panics are acceptable only for internal invariants (use `debug_assert!` where appropriate).
  - For unsafe code (rare), annotate with a short justification and expected invariants. Keep unsafe blocks minimal and well-tested.

- Performance & Memory
  - This project prioritizes low memory and high throughput. Avoid unnecessary allocations and copying.
  - Reuse buffers where feasible, prefer writing directly into buffers rather than building intermediate Strings.
  - Benchmarks and memory-sensitive code live in `benches/` and `examples/`; consult them before making large changes.

- Async & Concurrency
  - Public async APIs use `tokio` features when present; enable `cloud-*` features in `Cargo.toml` as required.
  - Prefer explicit `Send`/`Sync` bounds on types used across threads. Avoid hidden global mutable state.

- Documentation & Comments
  - Document public types and functions with `///` Rustdoc comments. Include short examples where useful.
  - Keep inline comments brief and focused; explain why a non-obvious implementation exists (not what it does).

- Tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KSD-CO/excelstream](https://github.com/KSD-CO/excelstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
