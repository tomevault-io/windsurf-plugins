---
trigger: always_on
description: <!-- markdownlint-disable -->
---

<!-- markdownlint-disable -->
<!-- markdown-link-check-disable -->

# Fluxion AI Coding Instructions

## Project Overview

Fluxion is a **Rust reactive streams library** with temporal ordering guarantees. It's a multi-runtime (Tokio/smol/async-std/WASM/Embassy), workspace-based project emphasizing correctness, zero unsafe code, and exceptional test coverage (990+ tests, 10.8:1 test-to-code ratio).

**Key Principle**: All stream operators preserve **temporal ordering** via timestamps - items are processed in timestamp order, not arrival order.

## Architecture & Workspace Structure

This is a **7-crate Cargo workspace** with clear separation of concerns:

```
fluxion-rx            → Convenience facade (re-exports all)
fluxion-core          → Core traits: Timestamped, HasTimestamp, StreamItem, FluxionError
fluxion-stream        → 27 stream operators (combine_latest, ordered_merge, etc.)
fluxion-stream-time   → Time-based operators (debounce, throttle, delay, sample, timeout)
fluxion-exec          → Execution: subscribe, subscribe_latest
fluxion-ordered-merge → Generic ordered merge algorithm
fluxion-test-utils    → Sequenced<T> wrapper, test fixtures, helpers
```

**Critical files**:
- [fluxion-core/src/timestamped.rs](../fluxion-core/src/timestamped.rs) - Core trait definitions
- [fluxion-stream/src/lib.rs](../fluxion-stream/src/lib.rs) - Operator extension traits
- [docs/FLUXION_OPERATOR_SUMMARY.md](../docs/FLUXION_OPERATOR_SUMMARY.md) - All 27 operators explained

## Core Traits & Patterns

### HasTimestamp vs Timestamped

**Two separate traits** for ordering (read [fluxion-core/src/has_timestamp.rs](../fluxion-core/src/has_timestamp.rs) and [fluxion-core/src/timestamped.rs](../fluxion-core/src/timestamped.rs)):

- `HasTimestamp` - Minimal read-only timestamp access (use for ordering)
- `Timestamped` - Extends HasTimestamp, adds `type Inner` + construction methods (use for wrapper types)

```rust
// Ordering-only (most common)
impl<T> HasTimestamp for MyType<T> {
    type Timestamp = u64;
    fn timestamp(&self) -> u64 { self.ts }
}

// Wrapper types with inner value
impl<T: Clone> Timestamped for MyWrapper<T> {
    type Inner = T;
    fn with_timestamp(value: T, ts: u64) -> Self { /* ... */ }
    fn into_inner(self) -> T { self.value }
}
```

### StreamItem<T> - Not Result<T, E>

**Never use `Result` in stream operators**. Use `StreamItem<T>`:

```rust
pub enum StreamItem<T> {
    Value(T),
    Error(FluxionError),
}
```

Errors **flow through streams** like data (no short-circuiting). Handle with `.on_error()` operator.

## Testing Standards (Non-Negotiable)

### Test File Organization

- **Integration tests only** in `tests/` directory (avoid `#[cfg(test)]` modules)
- One test file per operator: `tests/operator_name_tests.rs`
- Use `#[tokio::test]` for all async tests
- **Benchmarks** in `benches/` directory follow same comment standards as productive code

### Test Patterns

Always use `Sequenced<T>` from `fluxion-test-utils` for deterministic timestamps:

```rust
use fluxion_test_utils::{Sequenced, unwrap_stream, test_channel};

#[tokio::test]
async fn test_operator_behavior() {
    // Arrange
    let (tx, rx) = test_channel::<Sequenced<i32>>();
    let mut stream = rx.operator();

    // Act
    tx.try_send(Sequenced::with_timestamp(10, 1))?;
    tx.try_send(Sequenced::with_timestamp(20, 2))?;

    // Assert
    assert_eq!(unwrap_stream(&mut stream, 500).await.unwrap().value, expected);
}
```

**Test structure rules**:
- Structure tests with `// Arrange`, `// Act`, `// Assert` comments
- **No error messages in assertions** - assertions are self-documenting
- **No other comments** - test name and structure should be clear enough
- Use descriptive test function names: `test_broadcasts_to_multiple_subscribers`
- **Test file naming**: All test files must end with `_tests` (e.g., `stream_item_tests.rs`)
- **Multiple Act/Assert pairs allowed**: A test can have multiple `// Act` / `// Assert` pairs, but only one `// Arrange` section

**Never use `tokio::time::sleep()` in tests** - use `Sequenced<T>` for determinism.

## Development Workflows

### Building & Testing

```powershell
# Full CI check (recommended before commits)
.\.ci\ci.ps1

# Fast iteration
cargo test --workspace
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo fmt --all

# Runtime-specific tests
.\.ci\tokio_tests.ps1    # Tokio with nextest
.\.ci\wasm_tests.ps1     # WASM (requires Node.js)
.\.ci\smol_tests.ps1     # smol runtime
.\.ci\embassy_tests.ps1  # Embassy timers
```

**CI script locations**: `.ci/` directory contains all automation (see [.ci/ci.ps1](../.ci/ci.ps1))

### Runtime Selection & Features

**Default**: Tokio (zero config)
**Alternatives**: `runtime-smol`, `runtime-async-std`, `runtime-wasm`, `runtime-embassy`

```toml
# Switch runtimes
fluxion-rx = { version = "0.8.0", default-features = false, features = ["runtime-smol"] }
```

**Embassy notes**: Requires `no_std` + `alloc`, manual timer trait implementation. Only 24/27 operators work (subscribe_latest, partition, share incompatible due to static task allocation).

## Code Style Requirements

### Documentation

**Every public item must have docs** (no exceptions):

```rust
/// Brief one-line summary.
///
/// Detailed explanation with context and behavior guarantees.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umbgtt10/fluxion](https://github.com/umbgtt10/fluxion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
