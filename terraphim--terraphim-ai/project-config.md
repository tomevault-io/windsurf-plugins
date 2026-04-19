---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

You are an expert in Rust, async programming, and concurrent systems and WASM.

Key Principles
- Write clear, concise, and idiomatic Rust code with accurate examples.
- Use async programming paradigms effectively, leveraging `tokio` for concurrency.
- Prioritize modularity, clean code organization, and efficient resource management.
- Use expressive variable names that convey intent (e.g., `is_ready`, `has_data`).
- Adhere to Rust's naming conventions: snake_case for variables and functions, PascalCase for types and structs.
- Avoid code duplication; use functions and modules to encapsulate reusable logic.
- Write code with safety, concurrency, and performance in mind, embracing Rust's ownership and type system.

Async Programming
- Use `tokio` as the async runtime for handling asynchronous tasks and I/O.
- Implement async functions using `async fn` syntax.
- Leverage `tokio::spawn` for task spawning and concurrency.
- Use `tokio::select!` for managing multiple async tasks and cancellations.
- Favor structured concurrency: prefer scoped tasks and clean cancellation paths.
- Implement timeouts, retries, and backoff strategies for robust async operations.

Channels and Concurrency
- Use Rust's `tokio::sync::mpsc` for asynchronous, multi-producer, single-consumer channels.
- Use `tokio::sync::broadcast` for broadcasting messages to multiple consumers.
- Implement `tokio::sync::oneshot` for one-time communication between tasks.
- Prefer bounded channels for backpressure; handle capacity limits gracefully.
- Use `tokio::sync::Mutex` and `tokio::sync::RwLock` for shared state across tasks, avoiding deadlocks.

Error Handling and Safety
- Embrace Rust's Result and Option types for error handling.
- Use `?` operator to propagate errors in async functions.
- Implement custom error types using `thiserror` or `anyhow` for more descriptive errors.
- Handle errors and edge cases early, returning errors where appropriate.
- Use `.await` responsibly, ensuring safe points for context switching.

Testing
- Write unit tests with `tokio::test` for async tests.
- Use `tokio::time::pause` for testing time-dependent code without real delays.
- Implement integration tests to validate async behavior and concurrency.
- Never use mocks in tests.

## Testing Guidelines
- Keep fast unit tests inline with `mod tests {}`; put multi-crate checks in `tests/` or `test_*.sh`.
- Scope runs with `cargo test -p crate test`; add regression coverage for new failure modes.

## Rust Performance Practices
- Profile first (`cargo bench`, `cargo flamegraph`, `perf`) and land only measured wins.
- Borrow ripgrep tactics: reuse buffers with `with_capacity`, favor iterators, reach for `memchr`/SIMD, and hoist allocations out of loops.
- Apply inline directives sparingly—mark tiny wrappers `#[inline]`, keep cold errors `#[cold]`, and guard cleora-style `rayon::scope` loops with `#[inline(never)]`.
- Prefer zero-copy types (`&[u8]`, `bstr`) and parallelize CPU-bound graph work with `rayon`, feature-gated for graceful fallback.

## Commit & Pull Request Guidelines
- Use Conventional Commit prefixes (`fix:`, `feat:`, `refactor:`) and keep changes scoped.
- Ensure commits pass `cargo fmt`, `cargo clippy`, required `cargo test`, and desktop checks.
- PRs should explain motivation, link issues, list manual verification commands, and attach UI screenshots or logs when behavior shifts.

## Configuration & Security Tips
- Keep secrets in 1Password or `.env`. Use `build-env.sh` or `scripts/` helpers to bootstrap integrations.
- Wrap optional features (`openrouter`, `mcp-rust-sdk`) with graceful fallbacks for network failures.

Performance Optimization
- Minimize async overhead; use sync code where async is not needed.
- Use non-blocking operations and atomic data types for concurrency.
- Avoid blocking operations inside async functions; offload to dedicated blocking threads if necessary.
- Use `tokio::task::yield_now` to yield control in cooperative multitasking scenarios.
- Optimize data structures and algorithms for async use, reducing contention and lock duration.
- Use `tokio::time::sleep` and `tokio::time::interval` for efficient time-based operations.

Key Conventions
1. Structure the application into modules: separate concerns like networking, database, and business logic.
2. Use environment variables for configuration management (e.g., std crate).
3. Ensure code is well-documented with inline comments and Rustdoc.

Async Ecosystem
- Use `tokio` for async runtime and task management.
- Leverage `hyper` or `reqwest` for async HTTP requests.
- Use `serde` for serialization/deserialization.
- Use `sqlx` or `tokio-postgres` for async database interactions.
- Utilize `tonic` for gRPC with async support.
- use [salvo](https://salvo.rs/book/) for async web server and axum

## Important Rules

- **Never use sleep before curl** - Use proper wait mechanisms instead
- **Never use timeout command** - This command doesn't exist on macOS
- **Never use mocks in tests** - Use real implementations or integration tests

<!-- GITEA_PAGERANK_WORKFLOW_START -->


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/terraphim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
