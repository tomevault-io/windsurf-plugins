---
trigger: always_on
description: - This is a decentralized P2P open source social network protocol, replacing the likes of Twitter, Bluesky and Mastodon.
---

## Overview

- This is a decentralized P2P open source social network protocol, replacing the likes of Twitter, Bluesky and Mastodon.  
- Think a cryptographically secure, distributed posting system with Kademlia DHT for peer discovery.

### Key Design Patterns

- Completely decentralised
  - no central servers for things like account management, global counts, sending emails, etc.
  - no reliance on the goodwill of a single government or cloud provider or dns provider or repository hoster.
- Pluggable traits throughout
  - `TransportFactory`, `EnvironmentFactory`, `TimeProvider`, `ClientStorage`, `KeyLockerManager`
  - Swap implementations for testing vs production without changing protocol logic.
- Proof-of-Work (PoW), embedded in:
  - the RPC layer (`protocol/rpc.rs`), 
  - server identity (`tools/server_id.rs`), 
  - reporting feedback
- Cryptography stack
  - Signatures: Ed25519 (ed25519-dalek), with post-quantum ML-DSA and FN-DSA support
  - Hashing: Blake3 for general hashing, but then for PoW-hardness use chained Blake2, Blake3, SHA2, SHA3, Whirlpool, Groestl, Skein
  - Encryption: ChaCha20Poly1305
  - all hash libraries are built with `opt-level=3` even in debug mode (see workspace `Cargo.toml` profile overrides)

## Top level directives

- Be direct and not obseqious.  I don't need flattery.
- Prefer long variables names like are already in the codebase - generally prefer `encoded_post_bundle_feedback: EncodedPostBundleFeedbackV1` and `let bytes_gatherer: BytesGatherer = xxx` over `let g: BytesGatherer = xxx` 
- In git, omit "Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>" from commit messages
- Every suggested addition or refactor should be able to be tested using tests - write them if they are missing.  For a refactor, lets write any missing tests and test them before doing the refactor.
- All strings in both rust and typescript should be " delimited, not '

## Repository Layout

```
hashiverse/
├── hashiverse-rust/       # Rust workspace (server, protocol, WASM client)
├── hashiverse-client-web/ # TypeScript/React web client
├── hashiverse-rust/hashiverse-client-python/ # Python API wrapping hashiverse-client
├── www/                   # Static landing page assets
└── doc/                   # Documentation
```

## Build & Test Commands

### Rust (run from `hashiverse-rust/`)

The workspace `default-members` are the host-buildable crates (`hashiverse-lib`, `hashiverse-server-lib`, `hashiverse-server`, `hashiverse-client-rust`). The extension-only crates (`hashiverse-client-wasm`, `hashiverse-client-python`, `hashiverse-client-nodejs`) and the slow `hashiverse-integration-tests` are workspace members but excluded from the default selection. So bare `cargo` invocations (no `-p`, no `--workspace`) are safe and fast — they won't try to compile wasm-bindgen / pyo3-extension-module / napi against the host target.

```bash
# Fast: does everything compile? (default-members only, no tests run)
cargo check

# Default test run — host-buildable crates only
cargo nextest run --cargo-profile profiling

# Run tests for a specific crate
cargo nextest run --cargo-profile profiling -p hashiverse-lib
cargo nextest run --cargo-profile profiling -p hashiverse-server

# Slow integration tests — opt in explicitly; only worth running for major refactors
# Slow integration tests — opt in explicitly; only worth running for major refactors
cargo nextest run --cargo-profile profiling -p hashiverse-integration-tests

# Run a single test by name
cargo nextest run --cargo-profile profiling -p hashiverse-lib <test_name>

# WASM lib tests (wasmtime runner is configured in .cargo/config.toml)
cargo nextest run --target wasm32-wasip1 -p hashiverse-lib

# WASM client tests in headless Chrome (requires wasm-pack)
wasm-pack test --chrome --headless hashiverse-client-wasm --lib

# Python / Node extension crates compile only via their own build pipelines —
# not through bare cargo. See hashiverse-client-python/ and hashiverse-client-nodejs/
# READMEs for maturin / napi-cli usage.

# Build the default-members
cargo build

# Build absolutely everything in the workspace, including the extension crates
# (will fail on host unless you also pass --target wasm32-... for the wasm crate;
# only useful for sanity-checking workspace metadata, not for normal dev).
cargo build --workspace

# Run a simulation binary (starts 1 primary + 5 secondary servers + 10 clients)
cargo run -p hashiverse-integration-tests
```

**Toolchain**: Rust nightly (required — specified in `rust-toolchain.toml`).

**Lint**: No explicit lint commands configured; `cargo clippy` is standard.

**Formatting**: `rustfmt.toml` sets max line width to 250, tab spaces to 4.

### TypeScript Web Client (run from `hashiverse-client-web/`)

```bash
npm install          # Install dependencies
npm run dev          # Start dev server
npm run build        # Type-check + production build (rsbuild)
npm run preview      # Preview production build locally
npm run check        # Biome lint+format (writes fixes)
npm run format       # Biome format only (writes fixes)
```
**Stack**: React 19, TypeScript 5, Mantine 8 UI, Tiptap rich-text editor, React Router 7, rsbuild bundler, Biome linter/formatter.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hashiverse/hashiverse](https://github.com/hashiverse/hashiverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
