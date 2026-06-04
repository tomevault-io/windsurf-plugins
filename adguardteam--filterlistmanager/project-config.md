---
trigger: always_on
description: AdGuard Filter List Manager (FLM) is a Rust library for managing AdGuard filter
---

# AGENTS.md

## Project Overview

AdGuard Filter List Manager (FLM) is a Rust library for managing AdGuard filter
lists. It is used by different AdGuard applications (desktop, mobile) to integrate filter registries
([Filters Registry](https://github.com/AdguardTeam/FiltersRegistry),
[Hostlists Registry](https://github.com/AdguardTeam/HostlistsRegistry)),
check for updates, download them, implement differential updates, and more.

The library stores filter data in a local SQLite database and exposes its
functionality through a facade trait `FilterListManager`.

**This is a public open-source repository.** Never hardcode secrets, API keys,
or any sensitive credentials in the source code. Cryptographic constants like
derivation contexts and salts are fine — they are public protocol parameters,
not secrets.

## Repository Structure

```
crates/
├── filter-list-manager/   # Core library (crate: adguard-flm)
│   ├── src/
│   │   ├── lib.rs         # Public API re-exports
│   │   ├── manager/       # Main facade, models, update logic
│   │   ├── storage/       # SQLite DB layer: schema, migrations, repositories
│   │   ├── filters/       # Filter parsing, metadata extraction, preprocessor directives
│   │   ├── io/            # HTTP client, file I/O
│   │   └── utils/         # Shared utilities
│   └── resources/
│       └── sql/
│           ├── schema.sql          # Initial DB schema
│           └── migrations/         # Numbered SQL migration files (NNN-migration.sql)
├── ffi/                   # FFI wrapper (crate: adguard-flm-ffi)
│   ├── src/
│   │   ├── lib.rs                  # Thread-safe RwLock wrapper around FilterListManagerImpl
│   │   ├── native_interface/       # C ABI exports
│   │   ├── protobuf/               # .proto definitions for cross-language serialization
│   │   ├── protobuf_generated/     # Generated Rust code from .proto + conversion casts
│   │   └── platforms/              # Platform-specific builds (android/, apple/, windows/)
│   └── tests/
│       └── integration_test.rs
├── cli/                   # CLI tool (crate: adguard-flm-cli)
└── ffi-native-assets-generator/    # Helper for generating native assets
```

## Architecture

### Core Library (`adguard-flm`)

- **Facade pattern**: `FilterListManager` trait (`manager/mod.rs`) defines the
  public API. `FilterListManagerImpl` (`manager/filter_list_manager_impl.rs`)
  is the sole implementation.
- **Configuration**: `Configuration` struct
  (`manager/models/configuration/mod.rs`) holds all settings — DB path,
  locale, metadata URLs, proxy, expiration defaults, compilation policy, etc.
- **Filter types**: Index filters (from registry), Custom filters (user-added),
  Special filters (preconfigured by scripts). IDs for custom filters are
  negative (see `storage/constants.rs`).
- **Storage layer**: SQLite via `rusqlite`. Connection management through
  `DbConnectionManager`. Transactional helpers in `storage/mod.rs`.
- **Migrations**: Numbered SQL files in `resources/sql/migrations/`
  (`NNN-migration.sql`). Applied automatically by `storage/migrations.rs`.
  Schema version is tracked in the `metadata` table.
- **Filter parsing**: Metadata tag extraction (`! Title`, `! Expires`,
  `! Diff-Path`, etc.), preprocessor directives (`!#include`, `!#if/!#endif`),
  checksum validation — all in `filters/`.
- **HTTP**: `reqwest` with tokio runtime, supports proxy modes, gzip/deflate.
- **Error handling**: `FLMError` enum, `FLMResult<T>` type alias used
  throughout.

### FFI Layer (`adguard-flm-ffi`)

- Wraps `FilterListManagerImpl` in `RwLock` for thread safety.
- Uses **Protocol Buffers** (`prost`) for cross-language data serialization.
- `.proto` files are in `ffi/src/protobuf/`, generated code in
  `ffi/src/protobuf_generated/`.
- Conversion logic between FLM types and protobuf types is in
  `protobuf_generated/casts.rs`.
- Builds as `cdylib` + `staticlib` + `rlib`.
- Platform-specific wrappers: Android (Kotlin), Apple (Swift/ObjC), Windows
  (C#).

## Build & Development

- **Rust version**: 1.85+ (pinned in `rust-toolchain.toml`)
- **Workspace**: Cargo workspace with resolver v2
- **Key dependencies**: tokio, reqwest, rusqlite, serde, chrono, nom, prost
- **Features**:
  - `rusqlite-bundled` — bundles SQLite (useful for cross-compilation and tests)
  - `rustls-tls` — use rustls instead of native TLS

### Commands

| Action              | Command                                     |
|---------------------|---------------------------------------------|
| Build               | `cargo build --locked`                      |
| Run tests           | `cargo test --lib --locked`                 |
| Check formatting    | `cargo fmt --all -- --check`                |
| Lint                | `cargo clippy --locked`                     |
| Lint docs           | `npx markdownlint-cli .`                    |

### Clippy Configuration

- `allow-unwrap-in-tests = true`
- `too-many-arguments-threshold = 10`

## Versioning

- Crates `adguard-flm` and `adguard-flm-ffi` are versioned separately.
- Tags: `flm-${version}` for core, `ffi-${version}` for FFI.
- CI auto-increments patch versions on merge to master.
- See `CONTRIBUTING.md` for details.

## Critical Rules for Agents

### Database Schema Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdguardTeam/FilterListManager](https://github.com/AdguardTeam/FilterListManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
