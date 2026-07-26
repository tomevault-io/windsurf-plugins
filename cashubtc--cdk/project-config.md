---
trigger: always_on
description: Rust workspace (edition 2021) implementing the Cashu e-cash protocol.
---

# AGENTS.md - Cashu Development Kit (CDK)

Rust workspace (edition 2021) implementing the Cashu e-cash protocol.
24 crates in `crates/`, stable Rust (see `rust-toolchain.toml`), MSRV 1.85.0.

## Local Development Environment (Regtest)

CDK provides a complete regtest environment with Bitcoin, Lightning nodes, and pre-configured mints for end-to-end testing.

### Starting the Environment
```bash
# Enter the specialized shell with all dependencies
nix develop .#regtest

# Launch the full environment (Bitcoin + 4 LN nodes + 2 Mints)
just regtest
```
*Note: This launches `mprocs`. If running in a non-interactive environment, use `just regtest-status` to check health.*

### Interacting with Mints
- **CLN Mint:** `http://127.0.0.1:8085` (Env: `$CDK_TEST_MINT_URL`)
- **LND Mint:** `http://127.0.0.1:8087` (Env: `$CDK_TEST_MINT_URL_2`)

### Common Helper Commands
```bash
just mint-info       # Show both mints' keysets and info
just restart-mints   # Recompile and restart mints after code changes
just btc-mine 10     # Mine 10 blocks to confirm payments/open channels
just mint-test       # Run the full integration test suite against the environment
```

## Build / Check / Test / Lint Commands

```bash
# Build
cargo build --workspace --all-targets          # or: just build (alias: just b)
cargo check --workspace --all-targets          # or: just check (alias: just c)

# Lint
cargo fmt --all -- --check                     # format check
cargo clippy --workspace --all-targets -- -D warnings  # or: just clippy
typos                                          # spell checker

# Test - all unit tests (excludes postgres, needs running instance)
cargo test --lib --workspace --exclude cdk-postgres    # or: just test

# Test - single crate
cargo test -p cashu
cargo test -p cdk
cargo test -p cdk-common

# Test - single test function (use substring match)
cargo test -p cashu -- test_name_substring

# Test - single integration test file
cargo test -p cdk-integration-tests --test integration_tests_pure -- --test-threads 1
cargo test -p cdk-integration-tests --test test_swap_flow -- --test-threads 1
cargo test -p cdk-integration-tests --test wallet_saga -- --test-threads 1
cargo test -p cdk-integration-tests --test mint

# Test - pure integration tests with specific DB backend
CDK_TEST_DB_TYPE=memory cargo test -p cdk-integration-tests --test integration_tests_pure -- --test-threads 1
CDK_TEST_DB_TYPE=sqlite cargo test -p cdk-integration-tests --test test_swap_flow -- --test-threads 1

# Doc tests
cargo test --doc

# WASM check
cargo check -p cdk --target wasm32-unknown-unknown
```

## Workspace Lint Rules (Cargo.toml)

These are enforced workspace-wide:
- `unsafe_code = "forbid"` -- no unsafe code anywhere
- `unwrap_used = "deny"` -- never use `.unwrap()` in non-test code; use `?`, `.expect("reason")`, or pattern match
- `missing_docs = "warn"` -- all public items should have doc comments
- `missing_debug_implementations = "warn"`
- `missing_panics_doc = "warn"`
- `use_debug = "warn"` -- avoid `{:?}` in non-debug contexts

## Code Style

### Formatting (rustfmt.toml)

- 4-space indentation
- `imports_granularity = "Module"` -- merge imports from same module
- `group_imports = "StdExternalCrate"` -- blank lines between std/external/local groups
- Run `cargo fmt --all` before committing

### Import Order

```rust
// 1. core/std
use std::collections::HashMap;
use std::sync::Arc;

// 2. External crates
use serde::{Deserialize, Serialize};
use tokio::sync::Mutex;

// 3. Submodule declarations (if any)
mod x;
mod y;

// 4. Crate-internal imports
use crate::error::Error;
use super::something;
use self::x::Thing;       // always use self:: prefix for submodule imports
```

### Trait Bounds in `where` Clauses

```rust
// GOOD - bounds in where clause
fn new<N, T>(name: N, title: T) -> Self
where
    N: Into<String>,
    T: Into<String>,
{ ... }

// BAD - no inline bounds
fn new<N: Into<String>, T: Into<String>>(name: N, title: T) -> Self { ... }
```

### Use `Self` Over Type Name

In impl blocks, always use `Self` instead of repeating the type name.

### Derive Order

For public types, derive in this order: `Debug, Clone, Copy, PartialEq, Eq, Hash`.
Derive `Default` when a reasonable default exists.

### Logging / Tracing

Always use full path -- never import logging macros:
```rust
// GOOD
tracing::info!("Starting mint");
tracing::warn!("Connection lost: {}", reason);

// BAD
use tracing::warn;
warn!("Connection lost");
```

Exception: `use tracing::instrument;` is imported for the `#[instrument]` attribute.
Most public async methods should have `#[instrument(skip_all)]`.

### String Conversion

Prefer `.to_string()` / `.to_owned()` over `.into()` / `String::from()`.

### Control Flow

- Use `match` when both arms have logic; use `if let` only when one arm is empty.
- Prefer `match` over `if let ... else`.

### Modules

- Always use `mod x;` with a separate file, never inline `mod x { ... }`.
- Exception: `#[cfg(test)] mod tests { ... }` and `#[cfg(bench)] mod benches { ... }` are inline.

### fmt Imports

Import the module, not individual items:
```rust
use core::fmt;
impl fmt::Display for MyType { ... }
```

## Error Handling

- Define errors with `thiserror` (`#[derive(Debug, Error)]`), not `anyhow`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cashubtc/cdk](https://github.com/cashubtc/cdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
