---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kiwi is a Rust database targeting Redis 8.8.1 compatibility. It persists the complete authoritative data set with RocksDB, replicates with OpenRaft, and separates network I/O and storage into two distinct Tokio runtimes that communicate via async message channels.

The exact Redis compatibility and interface-design baseline is tag `8.8.1`, commit `77b6c308396c9700672390a210143a8496fb4b10`. Current required work runs Cache OFF and focuses on compatibility, authoritative RocksDB recovery, OpenRaft correctness, and system stability. The Embedded Redis Hot Tier remains design-only until the stability gate passes and the user explicitly authorizes a separate implementation task.

## Common Development Commands

Use `make` for day-to-day tasks. Build and test targets delegate to `scripts/dev.sh`, which automatically uses `sccache` when installed.

| Command | Purpose |
|---------|---------|
| `make check` | Fast syntax check (`cargo check`); preferred during iterative development. |
| `make build` | Debug build. |
| `make release` | Release build. |
| `make standalone` | Build and run a single-node server on the default port (`127.0.0.1:7379`). |
| `make cluster` | Start a local multi-node Raft cluster (`make cluster NODES=5`). |
| `make test` | Run all Rust unit tests. Sets `RUST_TEST_THREADS=1` and raises the fd limit. |
| `make fmt` / `make fmt-check` | Format code / check formatting (CI). |
| `make lint` | Run clippy with project lints (`-D warnings -D clippy::unwrap_used`). |
| `./scripts/dev.sh test --release` | Run tests in release mode. |
| `./scripts/dev.sh build --debug` | Build with full debug symbols; disables sccache. |

### Running a Single Test

```bash
# Run one test inside a specific crate
cargo test --package storage test_redis_mset

# Run by test name across the workspace
cargo test test_redis_mset
```

### Python Integration Tests

Requires a running server:

```bash
# Terminal 1
make standalone

# Terminal 2
make -C tests install-deps
make -C tests test-python
```

## Toolchain & Build Notes

- Normal development, CI, and release builds use Rust 1.97.1 stable. The root
  `rust-toolchain.toml` selects the exact toolchain automatically; verify it with
  `rustup show active-toolchain` and `rustc --version --verbose`.
- All Kiwi workspace crates use Rust 2024 Edition.
- Dated nightly toolchains are reserved for specialized checks such as
  Sanitizers and do not define the normal development baseline.
- The first build compiles `librocksdb-sys` from source and can take ~18 minutes. Incremental builds with `sccache` are typically 30 seconds–2 minutes.
- The project depends on a forked RocksDB crate (`arana-db/rust-rocksdb`) because upstream does not yet expose the `TablePropertiesCollector` FFI functions required by the Raft module. Do not switch to the official `rust-rocksdb` crate.
- `protoc` (protobuf compiler) is required. Windows builds use the Rust MSVC
  target and Visual Studio C++ build tools; Linux and macOS builds need the
  project's native C/C++ build dependencies.

## Architecture

### Crate Layout

Workspace members under `src/`:

- `server/` — Binary entry point (`kiwi`), `RuntimeManager` setup, and Raft wiring.
- `net/` — TCP/Unix server, connection handling, pipeline, storage client, and executor integration.
- `resp/` — RESP protocol parser, encoder, `RespData` types, and command negotiation.
- `cmd/` — Redis command implementations. Each command implements the `Cmd` trait.
- `executor/` — Async command executor / task pool.
- `client/` — Per-connection client state (`argv`, `cmd_name`, `key`, reply buffer, authentication).
- `storage/` — Multi-instance concrete RocksDB ownership, column families, TTL, key encoding, and log index for Raft.
- `raft/` — OpenRaft integration, concrete RocksDB log-store ownership, state machine, snapshot archive, and gRPC services.
- `conf/` — Configuration loading, validation, and sample-config generation.
- `kstd/` — Utilities, including `LockMgr` for sharded key-level locking.
- `common/runtime/` — Dual-runtime manager, async message channel between network and storage runtimes, and `StorageServer`.
- `common/macro/` — Proc macros, including `#[stack_trace_debug]`.

### Request Flow

```text
Client → TCP accept [network runtime] → RESP parse → command lookup
  → connection-local execution or executor_ext admission/dispatch
  → StorageClient → bounded async message channel
  → StorageServer [storage runtime] → Cmd.execute() → Storage/RocksDB
    ← oneshot response ←
  → RESP encode [network runtime] → write back to client
```

`CmdExecutor` is not the active production request queue on this path. Network
code performs the initial command admission, while `StorageServer` reconstructs
the execution context and invokes `Cmd::execute` on the storage runtime.

### Adding a Redis Command

Commands implement the `Cmd` trait in `src/cmd/src/lib.rs`:

- `meta()` → `CmdMeta` (name, arity, flags such as `WRITE`, `READONLY`, `RAFT`).
- `do_initial(&self, client)` → validate arguments and set the client key.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arana-db/kiwi](https://github.com/arana-db/kiwi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
