---
trigger: always_on
description: This playbook documents the expectations for production-grade Rust within the radkit agent SDK. It covers design principles, ownership discipline, safety, validation, and the workflow needed to ship confidently across native and WASM targets.
---

# radkit Rust Engineering Playbook

This playbook documents the expectations for production-grade Rust within the radkit agent SDK. It covers design principles, ownership discipline, safety, validation, and the workflow needed to ship confidently across native and WASM targets.

## Engineering Values
- Prefer clarity over cleverness. Readability, predictability, and explicit invariants take precedence over micro-optimizations.
- Embrace soundness. Design APIs so the compiler can enforce correctness with types, lifetimes, and traits.
- Minimize hidden state. Favor pure functions, explicit data flow, and narrow interfaces between modules.
- Guard portability. Ensure code paths work across Linux, macOS, Windows, and WASM unless a platform-specific feature is explicitly required.

## Project Configuration
- Declare the current `rust-version` (MSRV) in `Cargo.toml` and update it intentionally. All CI jobs must build and test against this MSRV.
- Use Rust 2021 edition unless a later stable edition is required; keep `rust-toolchain.toml` in sync with the workspace toolchain.
- At the crate root enable the strongest baseline lints we can sustain, e.g.
  ```rust
  #![deny(unsafe_code, unreachable_patterns, unused_must_use)]
  #![warn(clippy::all, clippy::pedantic, clippy::nursery)]
  ```
  Document any relaxed lint so reviewers know why it is needed.
- Keep `lib.rs`/`main.rs` dependency graphs under control by isolating optional functionality in feature-gated modules.

## API and Architecture Design
- Model domain concepts explicitly. Use `struct` and `enum` types with `#[non_exhaustive]` only when future-proofing is intentional.
- Prefer `trait`-driven design over type erasure. Implement conversion traits (`From`, `TryFrom`, `Into`) for ergonomic interop.
- Hide implementation detail behind `pub(crate)` modules. Expose the smallest viable public surface and document invariants via `///` comments.
- Keep constructors and builders validating their inputs; use smart constructors to guarantee invariants after instantiation.
- Separate synchronous, asynchronous, and WASM-specific APIs with feature flags or module boundaries to avoid mixing concerns.

## Ownership and Borrowing Discipline
- Default to borrowing (`&T`, `&mut T`) and slices (`&[T]`) in APIs; take ownership only when necessary. Return `&str`/`&[u8]` instead of allocating `String`/`Vec` when data lives long enough.
- Avoid `clone()` on hot paths. Reach for `Cow<'_, T>`, iterators, or reference-counted pointers (`Arc`, `Rc`) when sharing data.
- Design explicit lifetimes when returning references from structs. Prefer `Arc` + `Weak` for shared ownership with drop ordering requirements.
- Work with interior mutability (`Mutex`, `RwLock`, `parking_lot`, `RefCell`) only when borrowing rules cannot model the invariants. Document why interior mutability is required.
- Validate invariants inside `Drop` implementations and avoid surprising side effects; do not block or panic inside `Drop` paths.

## Async and Concurrency
- Restrict asynchronous APIs to types that are `Send + Sync` unless the lack of thread safety is explicitly documented.
- Never block the runtime. Route CPU intensive or blocking I/O work through `spawn_blocking` or dedicated worker threads.
- Propagate cancellation with `futures::select!`, `tokio::select!`, or cooperative checks so async tasks tear down quickly.
- Use `Arc` + `RwLock`/`Mutex` sparingly; prefer message passing (`mpsc`, `broadcast`, `watch`) or atomics for shared state.
- Validate cross-task ordering with tools like `loom` for tricky concurrency primitives; document assumptions about ordering and visibility.
- Ensure WASM code paths stay single-threaded: guard multi-threaded constructs with `#[cfg(not(target_family = "wasm"))]`.

## Error Handling
- Surface typed errors from library boundaries using `thiserror` or manual enums. Provide context with `anyhow::Context` or `eyre::WrapErr` inside leaf functions, but do not leak `anyhow::Error` across crate boundaries.
- Map external errors into our domain-specific variants early so upstream code can match on them predictably.
- Avoid panics except for programmer bugs (`debug_assert!`) or irrecoverable invariants. Audit `unwrap`/`expect` in PRs; justify any remaining usage in comments.
- Use `Result<T, E>` even for internal helpers if failure is possible. Prefer returning `Option<T>` only when absence is the only failure mode.

## Formatting, Lints, and Static Analysis
- Run `cargo fmt --all` before every commit; CI rejects formatting drift.
- Keep `cargo clippy --workspace --all-targets --all-features -D warnings` green. Document every `#[allow]` with a reason and a follow-up issue if needed.
- Periodically run `cargo fix --allow-dirty` to adopt new idioms introduced by compiler upgrades, but always review the diff manually.
- Use `cargo udeps` to remove unused dependencies and `cargo deny` or `cargo audit` to flag vulnerable or unlicensed crates as part of release pre-checks.

## Testing and Validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agents-sh/radkit](https://github.com/agents-sh/radkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
