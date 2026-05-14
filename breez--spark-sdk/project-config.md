---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Dependencies

The following system dependencies are required:

```bash
# Debian/Ubuntu
apt-get install -y protobuf-compiler

# macOS
brew install protobuf

# Arch Linux
pacman -S protobuf
```

## Build Commands

```bash
make build              # Build workspace (excludes WASM)
make build-release      # Release build with LTO
make build-wasm         # Build for WASM target
```

## Testing

```bash
make cargo-test         # Run Rust unit tests
make wasm-test          # Run WASM tests (browser + Node.js)
make itest              # Integration tests (requires Docker)
make breez-itest        # Breez integration tests (requires faucet credentials)
```

Run a single test:
```bash
cargo test <test_name> -p <package>
```

## Code Quality

```bash
make check              # Run all checks (fmt, clippy, tests) - use before committing
make fmt-check          # Check formatting
make fmt-fix            # Fix formatting
make clippy-check       # Run clippy lints (cargo + WASM)
make clippy-fix         # Fix clippy issues
```

## Architecture

### Crate Structure

- **crates/breez-sdk/core** - Main SDK library with public API (`BreezSdk`)
- **crates/breez-sdk/common** - Shared utilities, LNURL support, networking, sync protocol
- **crates/breez-sdk/bindings** - UniFFI bindings for Go, Kotlin, Python, React Native, Swift
- **crates/breez-sdk/wasm** - WebAssembly bindings for JavaScript/TypeScript
- **crates/breez-sdk/cli** - Command-line interface for testing
- **crates/spark** - Low-level Spark protocol (addresses, signing, operators, tokens)
- **crates/spark-wallet** - High-level wallet operations wrapping Spark protocol
- **crates/xtask** - Custom build tasks (powers `make` commands via `cargo xtask`)

### Key Abstractions

- `Storage` trait - Pluggable persistence layer (see Storage Implementations below)
- `Signer` trait - Cryptographic operations (FROST threshold signing)
- `BitcoinChainService` trait - Blockchain provider interface
- `EventEmitter` - Broadcasts `SdkEvent` (Synced, PaymentSucceeded, PaymentFailed, etc.)

### Storage Implementations

The `Storage` trait (`crates/breez-sdk/core/src/persist/mod.rs`) has multiple implementations. **When adding new Storage functionality, all implementations must be updated.**

| Implementation | Location | Platform | DB |
|---|---|---|---|
| SQLite (Rust) | `crates/breez-sdk/core/src/persist/sqlite.rs` | Native (macOS, Linux, Windows) | SQLite |
| PostgreSQL (Rust) | `crates/breez-sdk/core/src/persist/postgres.rs` | Server (feature-gated: `postgres`) | PostgreSQL |
| Web (JS) | `crates/breez-sdk/wasm/js/web-storage/index.js` | Browser (WASM) | IndexedDB |
| Node SQLite (JS) | `crates/breez-sdk/wasm/js/node-storage/index.cjs` | Node.js (WASM) | SQLite (`better-sqlite3`) |
| Node Postgres (JS) | `crates/breez-sdk/wasm/js/postgres-storage/index.cjs` | Node.js (WASM) | PostgreSQL (`pg`) |

All implementations run the **same shared test suite** in `crates/breez-sdk/core/src/persist/tests.rs`. When modifying storage:

1. Update every implementation listed above
2. Add test coverage to the shared test suite (`tests.rs`)
3. Add calls to any new test functions in **each** implementation's test harness:
   - Rust SQLite: `crates/breez-sdk/core/src/persist/sqlite.rs` (test module at bottom)
   - Rust Postgres: `crates/breez-sdk/core/src/persist/postgres.rs` (test module at bottom)
   - Web: `crates/breez-sdk/wasm/src/persist/tests/web.rs`
   - Node SQLite: `crates/breez-sdk/wasm/src/persist/tests/node.rs`
   - Node Postgres: `crates/breez-sdk/wasm/src/persist/tests/postgres.rs`

JS implementations also have migration files (`migrations.cjs`) alongside their `index.cjs`.

### Data Flow

```
BreezSdk (API) → SparkWallet (wallet ops) → Spark (protocol) → Operators (gRPC)
     ↓
Storage → SyncedStorage → Breez Sync Service (multi-device)
```

## Updating SDK Interfaces

When changing the SDK's public interface, update these files:

1. **crates/breez-sdk/core/src/models.rs** - Add UniFFI macros to interface types
2. **crates/breez-sdk/wasm/src/models.rs** - Update exported structs/enums
3. **crates/breez-sdk/wasm/src/sdk.rs** - Update WASM interface
4. **packages/flutter/rust/src/models.rs** - Update mirrored structs/enums
5. **packages/flutter/rust/src/sdk.rs** - Update Flutter interface

## Documentation Inline Syntax

When writing mdbook documentation in `docs/breez-sdk/src/`, use these preprocessor macros for language-aware inline code that adapts to the selected language tab:

- `{{#name identifier}}` - For functions, methods, parameters, properties
  - Rust/Python: `get_info` (snake_case)
  - Swift/Kotlin/JS/Flutter: `getInfo` (camelCase)
  - Go/C#: `GetInfo` (PascalCase)

- `{{#enum Type::Variant}}` - For enum variants
  - Rust: `SdkEvent::Synced`
  - Python: `SdkEvent.SYNCED`
  - Swift: `SdkEvent.synced`
  - Go: `SdkEventSynced`
  - Others: `SdkEvent.Synced`

Example:
```markdown
Call {{#name get_info}} after each {{#enum SdkEvent::Synced}} event.
```

See [snippets-processor/src/main.rs](docs/breez-sdk/snippets-processor/src/main.rs) for transformation rules.

## CLI Modification Policy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breez/spark-sdk](https://github.com/breez/spark-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
