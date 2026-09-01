---
trigger: always_on
description: IMPORTANT: Never enter plan mode automatically!!! Never enter plan mode automatically!!!
---

# sbe

IMPORTANT: Never enter plan mode automatically!!! Never enter plan mode automatically!!!

## Core Principles

### Code Quality

- **SOLID and DRY Principles**: Maintain clean, maintainable code following SOLID and DRY principles
- **No Incomplete Code**: Never write TODO comments or temporary solutions. If you encounter such a situation:
  1. Stop the current task
  2. Review the problem globally
  3. Rethink the design and identify the best alternative solutions
  4. Proceed with the complete solution
- **Thorough Analysis**: Always perform a comprehensive review and analysis of the problem before starting work
- Do not suppress dead code, remove them; Unless explicitly requested, do not go through deprecation process, just remove the code that is no longer needed.

### Development Workflow

- **Latest Dependencies**: Always search the web for the latest dependencies or helm charts or resources and their current usage patterns. If doing a deep research, put the research doc under ./docs/research. You shall look into that directory before doing researches.
- **Automation via Makefile**:
  - Explore existing Makefile targets and use them accordingly
  - For new automation tasks, always add a Makefile target instead of creating shell scripts
  - Keep automation consistent and discoverable

## Documentation

For specs, explore ./specs directory and put it to the right place, name the spec file as {feature-name}-{type}.md and update index.md accordingly. type can be prd, design, impl-plan, verification-plan, review, etc.

For docs, explore ./docs directory and put it to the right place, and update index.md accordingly. If you generate documentation that wasn't explicitly requested, make sure to place it under `./docs` and follow the same rule.

## Toolchain & Build

- Always use Rust 2024 edition with latest stable version. Pin version in `rust-toolchain.toml`.
- Always run `cargo build`, `cargo test`, `cargo +nightly fmt`, and `cargo clippy -- -D warnings` before finishing the task.
- Use `cargo clippy -- -D warnings -W clippy::pedantic` for stricter linting. Allow specific lints with justification.
- Run `cargo audit` regularly to check for security vulnerabilities in dependencies.
- Use `cargo-deny` to enforce license policies and ban specific crates.
- Enable all rustc lints in Cargo.toml: `#![warn(rust_2024_compatibility, missing_docs, missing_debug_implementations)]`.
- DO NOT use `cargo clean` at any time. If you indeed need it, ask user for permission

## Error Handling

- Never use `unwrap()` or `expect()` in production code. Always handle errors properly with `?` operator or explicit match.
- Use `thiserror` for library error types (with custom error enums). Use `anyhow` for application error handling.
- Implement proper error context with `.context()` or `.with_context()` when propagating errors.
- Use `Result<T>` as return type for fallible functions. Never use `Option` to represent errors.
- For unrecoverable errors in applications, use `panic!`. For libraries, always return `Result`.
- Define domain-specific error types using enums with `thiserror`. Include source errors with `#[source]`.

## Async & Concurrency

- Use Tokio as async runtime. Always specify features explicitly (e.g., `tokio = { version = "1", features = ["rt-multi-thread", "macros"] }`).
- Prefer message passing (channels) over shared state. Use `tokio::sync::mpsc` for MPSC, `flume` for faster channels.
- Organize system into subsystems using Actor model. Each actor owns its state and communicates via channels. For non-Send/Sync types (e.g., Tantivy Index), isolate in dedicated thread and use channels for communication. Never wrap in Mutex/RwLock. For Actors, it shall have proper start/stop/restart logic. Consider using AtomicBool for shutdown signal.
- Use `DashMap` for concurrent HashMap instead of `Mutex<HashMap>` or `RwLock<HashMap>`. Provides better performance.
- Use `ArcSwap` for infrequently updated shared data (e.g., configuration). Allows lock-free reads.
- Always consider using config crate for configuration management. Always use yaml format for configuration. For data that shall be tuned at runtime, put in configuration file. For data that shall be tuned at compile time, use compile time constants.
- For async traits, use native `async fn` in traits (stable since Rust 1.75). **Exception**: When traits require object safety (used with `dyn Trait` for dynamic dispatch like `Arc<dyn TaskStorage>`), use `async-trait` crate and document the reason in module-level docs.
- Always handle task panics. Use `tokio::spawn` with proper error handling. Consider `tokio::task::JoinSet` for managing multiple tasks.
- Avoid blocking operations in async contexts. Use `tokio::task::spawn_blocking` for CPU-intensive or blocking operations.
- Use structured concurrency patterns. Ensure spawned tasks are awaited or explicitly detached with justification.

## Type Design & API

- Use `typed-builder` crate for builder pattern on structs with more than 5 fields. Provides compile-time guarantees. Use plain `new()` for simple constructors with few arguments.
- Make types as specific as possible. Prefer `NonZeroU32` over `u32` when zero is invalid.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyrchen/sbe](https://github.com/tyrchen/sbe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
