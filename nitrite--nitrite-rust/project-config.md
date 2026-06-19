---
trigger: always_on
description: This file provides a complete, accurate reference for AI coding agents (Codex, Claude Code, Gemini, Windsurf, etc.) working in the `nitrite-rust` workspace.
---

# Nitrite Rust — Agent SKILL File

This file provides a complete, accurate reference for AI coding agents (Codex, Claude Code, Gemini, Windsurf, etc.) working in the `nitrite-rust` workspace.

---

## Project Overview

**Nitrite** is an embedded, serverless NoSQL document database written in Rust. It stores documents in typed collections, supports ACID transactions, pluggable storage backends, spatial indexing, and full-text search.

- **Version**: `0.2.0`
- **Edition**: Rust 2021
- **License**: Apache-2.0
- **MSRV**: Rust 1.70+
- **Repository**: https://github.com/nitrite/nitrite-rust

---

## Workspace Layout

```
nitrite-rust/
├── nitrite/                  # Core database engine (published crate)
├── nitrite-derive/           # Procedural macros: Convertible, NitriteEntity
├── nitrite-fjall-adapter/    # Persistent storage via Fjall LSM-tree (published)
├── nitrite-spatial/          # R-tree spatial indexing (published)
├── nitrite-tantivy-fts/      # Full-text search via Tantivy (published)
├── nitrite-int-test/         # Integration tests (publish = false)
├── nitrite-bench/            # Criterion benchmarks (publish = false)
└── Cargo.toml                # Workspace root, resolver = "2"
```

Workspace-shared dependency: `parking_lot = "0.12.3"`.

---

## Build & Test Commands

```bash
# Build everything
cargo build --workspace

# Run all unit + doc tests
cargo test --workspace --verbose

# Run tests with custom separator feature flag
cargo test --features custom_separator -- custom_separator_test

# Integration tests: Fjall (persistent) backend (default)
cargo test -p nitrite_int_test

# Integration tests: in-memory backend
cargo test -p nitrite_int_test --features memory --no-default-features

# Run benchmarks
cargo bench -p nitrite_bench

# Run comparison benchmarks (vs SQLite/Redb/Sled)
cargo bench -p nitrite_bench --features comparison
```

CI runs on `ubuntu-latest`, `macos-latest`, and `windows-latest` via `.github/workflows/rust.yml`.

---

## Crate-by-Crate Reference

### `nitrite` — Core Engine

**Crate name**: `nitrite`
**Entry point**: `nitrite::nitrite::Nitrite` (PIMPL pattern over `Arc<NitriteInner>`)

#### Key Public Modules

| Module | Key Exports |
|--------|-------------|
| `nitrite::nitrite` | `Nitrite` — database handle |
| `nitrite::nitrite_builder` | `NitriteBuilder` — fluent builder |
| `nitrite::nitrite_config` | `NitriteConfig` — config access |
| `nitrite::collection` | `Document`, `NitriteCollection`, `NitriteId`, `FindOptions`, `UpdateOptions`, `CollectionEvents`, `CollectionEventInfo`, `WriteResult` |
| `nitrite::filter` | `field()`, `all()`, `by_id()`, `and()`, `or()`, `not()` — filter API |
| `nitrite::index` | `IndexOptions`, `IndexDescriptor`, `unique_index()`, `non_unique_index()`, `full_text_index()`, `NitriteIndexer` |
| `nitrite::repository` | `ObjectRepository`, `NitriteEntity`, `RepositoryCursor` |
| `nitrite::transaction` | `Session`, `NitriteTransaction`, `TransactionContext`, `TransactionStore`, `TransactionalMap` |
| `nitrite::store` | `NitriteStore`, `NitriteMapProvider`, `NitriteStoreProvider`, `InMemoryStoreModule`, `StoreEventListener` |
| `nitrite::migration` | `Migration`, `MigrationStep`, `MigrationArguments`, `MigrationManager` |
| `nitrite::common` | `Value`, `Convertible`, `NitriteModule`, `NitritePlugin`, `NitritePluginProvider`, `PluginRegistrar`, `PersistentCollection`, `EventAware`, `AttributeAware`, `SortOrder`, `SortableFields`, `DocumentCursor`, `Processor`, constants |
| `nitrite::errors` | `NitriteError`, `NitriteResult<T>`, `ErrorKind` |
| `nitrite::metadata` | `NitriteMetadata` |

#### Global Statics (in `lib.rs`)

- `FIELD_SEPARATOR: LazyLock<Atomic<String>>` — default `"."`, configurable once via builder
- `ID_GENERATOR: LazyLock<SnowflakeIdGenerator>` — Snowflake-based unique IDs
- `SCHEDULER: LazyLock<Scheduler>` — background task scheduler

#### Features

- `default = ["serde"]` — enables serde support on `Value` and `Document`
- `custom_separator` — unlocks tests for non-default field separator
- `serde` — optional, enables serde derive impls for `Value`/`Document`

#### Key Dependencies

| Dependency | Version | Purpose |
|-----------|---------|---------|
| `im` | 15.1.0 | Persistent immutable `OrdMap` for `Document` |
| `parking_lot` | 0.12.3 | Fast mutexes/rwlocks |
| `dashmap` | 6.1.0 | Concurrent hash maps |
| `crossbeam-skiplist` | 0.1.3 | Ordered concurrent structures |
| `argon2` | 0.5.3 | Password hashing for auth |
| `aes-gcm` | 0.10.3 | AES-GCM encryption |
| `icu_collator` | 2.0.0 | Unicode-aware collation for sorting |
| `chrono` | 0.4.39 | Date/time support |
| `rand` | =0.8.5 | **Pinned** — CryptoRng trait compat issue |
| `basu` | 0.1.5 | Event bus system |
| `regex` | 1.11.1 | Regex filter support |

---

### `nitrite_derive` — Procedural Macros

**Crate name**: `nitrite_derive`
**Type**: `proc-macro` crate

Two derive macros:

```rust
#[derive(Convertible)]           // structs + enums
#[derive(NitriteEntity)]         // structs only
```

#### `Convertible`

- Works on **structs with named fields** and **enums**
- Unions are not supported

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nitrite/nitrite-rust](https://github.com/nitrite/nitrite-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
