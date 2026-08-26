---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

PrismDB is a single-node OLTP **multi-model** database engine written in Rust. Relational SQL tables, JSON-like documents, and ordered key-value pairs live on **one** storage engine, sharing one buffer pool, one WAL, and one transaction manager, so a single transaction can atomically mutate all three models.

Workspace version `0.2.0`, Apache-2.0, 17 crates under `crates/`, 7 standalone client SDKs under `sdks/`.

Binaries: `prismd` (server), `prism-shell` (REPL), `prism-fsck`, `prism-dump` - all defined in `crates/prism-cli/src/bin/`.

Explicitly out of scope for v1: distribution/replication, columnar analytics, and Postgres/Mongo/Redis wire compatibility. Do not add these without an ADR.

## Toolchain

Rust **1.85.0** exactly, pinned in `rust-toolchain.toml` (edition 2024, MSRV 1.85). Components: `clippy`, `rustfmt`.

Changing the pinned toolchain or the MSRV **requires an ADR**. The `sqlparser` dependency is deliberately configured with `default-features = false` because its `recursive-protection` feature pulls a build-dep needing Rust > 1.85 - do not "fix" this by enabling default features.

## Commands

Verify with these before declaring work done. CI sets `RUSTFLAGS: "-D warnings"`, so warnings fail the build.

```sh
cargo fmt --all                                                   # format (CI checks --check)
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-targets --all-features --locked
cargo deny check                                                  # advisories, licenses, sources
```

Two aliases exist in `.cargo/config.toml`:

```sh
cargo lint    # clippy --all-targets --all-features -- -D warnings
cargo ci      # test --all-targets --all-features --locked
```

Narrower loops while iterating:

```sh
cargo check                          # fast type-check
cargo test -p prism-wal              # one crate
cargo test -p prism-server --test durability   # one integration test file
cargo test -- --nocapture            # show stdout
cargo doc --no-deps --open           # API docs
```

CI (`.github/workflows/ci.yml`) runs on push to `main` and all PRs: an `fmt` job, a `test` job matrixed over ubuntu/macos/windows (clippy + test), and a `deny` job. All three OSes must pass - avoid Unix-only path or syscall assumptions.

### SDKs

Each SDK is a **standalone reimplementation of the wire protocol** - no FFI, no shared codegen, no native addon. Changing the protocol means updating `crates/prism-protocol` **and all seven SDKs**. Each is built from its own directory:

| SDK | Commands |
| --- | --- |
| `sdks/c`, `sdks/cpp` | `make`, `make test`, `make example`, `make clean` |
| `sdks/node` | `npm ci`, `npm run build`, `npm test`, `npm run example` (Node 22+ to run tests; published SDK supports Node ≥ 20) |
| `sdks/python` | setuptools build, Python ≥ 3.8; single test file `tests/test_codec.py` |
| `sdks/java` | `mvn` per `pom.xml` |
| `sdks/dotnet` | `dotnet` against `PrismDb.slnx` |
| `sdks/php` | `composer` per `composer.json` |

SDK tests are offline codec round-trips with hardcoded byte layouts, so a wire-format change will surface there. There is no cross-SDK conformance harness.

## Architecture: the layering rule

Crate dependencies flow **downward only**, never sideways between engines, never upward, no cycles. This is normative, not advisory - see `docs/architecture/module-layout.md`.

```
clients:  prism-shell   prism-client   prism-sdk-node   prism-cli
             ↓
server:   prism-server            (query dispatch + network)
             ↓
engines:  prism-sql   prism-doc   prism-kv     ← siblings, never depend on each other
             ↓
access:   prism-index             (B+tree, hash)
             ↓
core:     prism-core              (txn manager, MVCC, locks, ARIES recovery, record store)
             ↓
          prism-wal   prism-buffer
             ↓
          prism-storage           (disk manager; no Prism deps)

prism-protocol sits outside the stack: wire format only, shared by server and clients.
```

Four forbidden patterns:

1. `prism-storage` referencing the WAL.
2. `prism-core` referencing any engine.
3. Any engine depending on another engine.
4. Any crate referencing `prism-server` - except `prism-bench`.

The record store, transaction manager, buffer pool, and WAL are **singletons per process**, shared by all three access methods. That sharing is what makes cross-model transactions atomic; preserve it.

### Crate responsibilities

| Crate | Role |
| --- | --- |
| `prism-storage` | Page-based disk manager, checksums, DB header |
| `prism-wal` | Physiological ARIES write-ahead log, segments, records |
| `prism-buffer` | Clock-sweep buffer pool |
| `prism-core` | Transactions, MVCC visibility, lock manager, recovery, record store |
| `prism-index` | Access methods: B+tree today, extendible hash index still to come |
| `prism-sql` | SQL engine, catalog, type system |
| `prism-doc` | Document engine and value model |
| `prism-kv` | Ordered key-value engine |
| `prism-protocol` | Binary wire protocol codec, frames, messages |
| `prism-server` | Dispatcher, network server, auth, authz, TLS, audit, dump |
| `prism-client` | Rust client |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HafizMMoaz/prism-db](https://github.com/HafizMMoaz/prism-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
