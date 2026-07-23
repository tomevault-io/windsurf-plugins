---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Walrus is a decentralized storage protocol built on Sui. It uses Reed-Solomon erasure coding to
distribute blob data across a network of storage nodes, with smart contracts on Sui managing
registration, certification, and economics.

## Architecture

### Three-Layer Design

1. **Client Layer** (`walrus-sdk`): Encodes blobs into slivers via erasure coding, orchestrates
   parallel uploads to storage nodes, and interacts with Sui contracts for registration/certification.
2. **Storage Node Layer** (`walrus-service`): Axum-based REST API servers that store slivers in
   RocksDB, process blob events from Sui, manage epoch transitions, and participate in shard sync.
3. **Blockchain Layer** (`contracts/`): Move smart contracts on Sui managing blob lifecycle
   (register, certify, delete), committee state, staking, and WAL token economics.

### Blob Upload Flow

1. `BlobEncoder` splits raw data into primary/secondary sliver pairs using RS2 erasure coding
2. Merkle tree over sliver pairs produces the blob ID (committed on-chain)
3. `DistributedUploader` sends slivers in parallel to storage nodes based on committee shard assignments
4. Client calls Sui contract to register blob; nodes see `BlobRegistered` event and finalize storage
5. After quorum confirmation, client can certify the blob on-chain

### Epoch & Committee Model

- `ActiveCommittees` tracks current, previous, and next epoch committees
- Epoch changes reshuffle shard-to-node assignments; `EpochChangeDriver` manages transitions
- During transitions, both current and previous committees serve reads

### Key Crate Relationships

- **walrus-sdk** → uses **walrus-storage-node-client** (HTTP client to nodes) + **walrus-sui** (chain interactions)
- **walrus-service** → storage node binary + publisher/aggregator daemon; uses **walrus-core** (encoding/types) + **walrus-sui** (event processing)
- **walrus-core** → encoding primitives, cryptographic types, slivers, metadata (no network/chain deps)
- **walrus-upload-relay** → alternative upload path where clients pay tips; validates transactions and delegates to SDK
- **walrus-proxy** → metrics aggregation relay for Prometheus/Mimir
- **walrus-stress** → stress testing workloads against Walrus clusters
- **walrus-indexer** → indexes Walrus on-chain data

## Build & Development Commands

### Prerequisites

System packages: `libssl-dev pkg-config zlib1g-dev libpq-dev build-essential cmake`
Rust toolchain: 1.96 (specified in `rust-toolchain.toml`)

### Building

```bash
cargo build                          # Debug build
cargo build --release                # Release build
```

### Running Tests

```bash
# Run all tests (uses nextest)
cargo nextest run

# Run a single test by name
cargo nextest run my_test_name

# Run tests in a specific crate
cargo nextest run -p walrus-core

# Run ignored tests (require external Sui cluster)
cargo nextest run --run-ignored ignored-only

# Run doctests (nextest doesn't run these)
cargo test --doc
```

### Linting & Formatting

```bash
# Format (note: custom rustfmt config via CLI args, not rustfmt.toml)
cargo fmt -- --config group_imports=StdExternalCrate,imports_granularity=Crate,imports_layout=HorizontalVertical

# Clippy
cargo clippy --all-features --tests -- -D warnings

# All pre-commit hooks
prek run --all-files # or pre-commit run --all-files if using pre-commit
```

### Simulation Tests (msim)

Deterministic simulation tests for distributed system scenarios. Uses a patched Rust stdlib for
controlled scheduling and networking.

```bash
# One-time install
./scripts/simtest/install.sh

# Run all simtests
MSIM_TEST_SEED=1 cargo simtest simtest --profile simtest

# Run a specific simtest
MSIM_TEST_SEED=1 cargo simtest simtest <test_name> --profile simtest

# Run all seeds (CI runs seeds 1-5)
./scripts/run-all-simtests.sh
```

Key env vars: `MSIM_TEST_SEED` (default: 1), `WALRUS_GRPC_MIGRATION_LEVEL` (default: 100; set to 0 for legacy JSON-RPC).

### Move Contracts

```bash
# Run all Move tests
./scripts/move_tests.sh

# With coverage (minimum 70%)
./scripts/move_tests.sh -c

# Specific directory
./scripts/move_tests.sh -d testnet-contracts

# Manual (from contract dir)
sui move test --allow-dirty -e testnet
```

The correct `sui` binary version must match the Sui tag in `Cargo.toml` (currently `testnet-v1.66.1`).

### Binaries

```bash
cargo run --bin walrus -- <args>        # Client CLI
cargo run --bin walrus-node -- <args>   # Storage node
```

### Local Testbed

```bash
./scripts/local-testbed.sh              # Start local network (4 nodes, 10 shards)
./scripts/local-testbed.sh -c 8 -s 20   # Custom committee size and shards
```

## Code Conventions

### Error Handling
- No `unwrap()` in production code (only tests); use `expect()` with explanation
- Prefer explicit `Result` types

### Type Conversions
- No `as` casts on numeric types (clippy `cast_possible_truncation` is enforced)
- Use `from`/`into` or `try_from`/`try_into`

### Logging
- `tracing` crate; lowercase messages, no trailing period
- Structured fields over string interpolation: `tracing::info!(blob_id = %id, "stored blob")`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MystenLabs/walrus](https://github.com/MystenLabs/walrus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
