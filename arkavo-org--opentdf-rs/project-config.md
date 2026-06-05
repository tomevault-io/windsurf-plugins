---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important Project Files
When starting a new conversation or initializing, please read these files:
- `README.md` - Project overview and basic usage
- `CLAUDE.md` - Coding guidelines (this file)
- `REQUIREMENTS.md` - Technical and functional requirements

## Build/Test Commands

### Core Library
- Build: `cargo build`
- Build all features: `cargo build --all-features`
- Format code: `cargo fmt --all`
- Check formatting: `cargo fmt --all --check`
- Run all tests: `cargo test`
- Run tests with all features: `cargo test --all-features`
- Run a single test: `cargo test test_name`
- Run tests with output: `cargo test -- --nocapture`
- Run clippy lints: `cargo clippy`
- Run clippy with all features: `cargo clippy --all-targets --all-features`
- Fix all clippy warnings: `cargo clippy --fix`
- Build with warnings as errors: `RUSTFLAGS="-D warnings" cargo build`
- Run clippy with warnings as errors: `cargo clippy --all-targets --all-features -- -D warnings`

### WASM
- Build WASM (web): `wasm-pack build --target web --out-dir pkg-web` (from `crates/wasm/`)
- Build WASM (bundler): `wasm-pack build --target bundler --out-dir pkg` (from `crates/wasm/`)
- Build WASM (nodejs): `wasm-pack build --target nodejs --out-dir pkg-node` (from `crates/wasm/`)
- Test WASM builds for all targets before committing

## Architecture Overview

### Workspace Structure
This is a Cargo workspace with the following crates:
- **`opentdf`**: Main library (root crate)
- **`opentdf-protocol`**: Protocol types and structures (no I/O, pure data)
- **`opentdf-crypto`**: Cryptographic operations (KEM, encryption, hashing)
- **`opentdf-wasm`**: WebAssembly bindings for browser use

### Core Modules
- **`src/lib.rs`**: Main library entry point, exports public API
- **`src/archive.rs`**: TDF archive creation and reading using ZIP format
  - `TdfArchive`: Read TDF archives from files or streams
  - `TdfArchiveBuilder`: Create new TDF archives with manifest and payload
- **`src/kas.rs`**: KAS (Key Access Service) client for key rewrap protocol
  - `KasClient`: Async HTTP client using reqwest + tokio
  - JWT signing with RS256 for authentication
  - Full KAS v2 rewrap protocol implementation
- **`src/manifest.rs`**: TDF manifest structure and serialization
  - `TdfManifest`: JSON manifest containing encryption metadata and policy
- **`src/policy.rs`**: Attribute-Based Access Control (ABAC) policy system
  - `AttributeIdentifier`: Namespace-qualified attribute names
  - `AttributeValue`: Type-safe attribute values (string, number, boolean, datetime, arrays)
  - `Operator`: Rich comparison operators (equals, contains, in, minimumOf, etc.)
  - `AttributePolicy`: Logical policy expressions with AND/OR/NOT
  - `Policy`: Complete policy with time constraints and dissemination

### Protocol Crate (`crates/protocol`)
- Pure data structures, no I/O operations
- Shared types between native and WASM
- KAS protocol request/response types
- WASM-compatible (uses `uuid` with "js" feature on wasm32)

### Crypto Crate (`crates/crypto`)
- AES-256-GCM encryption/decryption
- RSA-OAEP key encapsulation (SHA-1 for Go SDK compatibility, SHA-256 recommended)
- HMAC-SHA256 for policy binding
- Modular KEM traits for extensibility

### WASM Crate (`crates/wasm`)
- Browser-compatible bindings using wasm-bindgen
- Uses `opentdf` with `default-features = false` (no tokio)
- Implements KAS client using web_sys Fetch API
- Shares protocol structs with native implementation
- Supports all TDF operations in-browser (encrypt, decrypt, policy evaluation)

### Key Design Patterns
- **Encryption Flow**: Data → AES-256-GCM encryption → Policy binding via HMAC-SHA256 → ZIP archive
- **Policy Evaluation**: User attributes → Policy tree evaluation → Access decision with audit trail
- **Type Safety**: Strong typing with `thiserror` for error handling, `serde` for serialization

## Code Style Guidelines

- **Formatting**: Use `rustfmt` defaults (enforced via `cargo fmt`)
- **Error Handling**: Use `thiserror` for custom error types (see `PolicyError`, `EncryptionError`, `TdfError`)
- **Naming**: Follow Rust conventions - snake_case for variables/functions, CamelCase for types
- **Imports**: Group std imports first, then external crates, then local modules
- **Types**: Use strong typing and prefer specific error types over `Box<dyn Error>`
- **Documentation**: Document public API with doc comments
- **Testing**: Write unit tests for each module, integration tests for API
- **Error Types**: Create enum-based error types that implement `std::error::Error`
- **Serialization**: Use `serde` attributes consistently for JSON serialization
- **Security**: Never commit secrets, validate cryptographic operations
- **Commit Preparation**: Always run `cargo build`, `cargo clippy`, and `cargo fmt` before committing
- **Dead Code**: Remove unused code rather than using `#[allow(dead_code)]` attributes
- **Warnings**: Eliminate all compiler warnings before committing code

## Workflow Guidelines

- Always run the full test suite before submitting PRs: `cargo test`
- Address all clippy warnings before submitting code: `cargo clippy`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkavo-org/opentdf-rs](https://github.com/arkavo-org/opentdf-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
