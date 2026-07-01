---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Alpenglow is a research reference implementation of the Alpenglow consensus protocol - a high-performance, global Proof-of-Stake blockchain consensus system with erasure coding for data availability. The project is written in Rust and designed for distributed systems research.

## Essential Commands

Dev tasks run through a `Justfile` (install with `cargo install just`; run `just`
to list recipes, `just setup` once per machine to install the toolchain). The
old `*.sh` scripts are gone — only `run.sh` and `download_data.sh` remain.

### Build and Run
```bash
cargo build --release                          # Build in release mode
./run.sh                                       # Run local cluster (alpenglow=debug)
RUST_LOG="alpenglow=debug" cargo run --release --bin=local_cluster
cargo run --release --bin=simulations          # Run protocol simulations
```
Other binaries: `node`, `all2all_test`, `performance_test`, `workload_generator`.

### Testing
Uses `cargo-nextest` (not `cargo test`):
```bash
just test                    # Fast tests (default), all targets/features
just test-doc                # Doctests (nextest doesn't run these)
just test-smoke              # Ignored-by-default smoke tests, release mode
just test-sequential         # Perf-sensitive tests that must run with --jobs=1
just test-ci                 # Full CI suite: test + test-doc + test-smoke + test-sequential
just test-slow               # Full slow suite (~10 min)
just test-many               # Run fast+sequential 50x to surface flaky tests
cargo nextest run <name>     # Run a specific test directly
```

### Linting and Quality
```bash
just check                   # Full local CI: fmt, clippy, build, doc, deny, machete, typos, fuzz-build, test-ci
just fmt                     # cargo +nightly fmt --all -- --check
just clippy                  # cargo clippy --all-targets --all-features -- -D warnings
just doc                     # cargo doc with -D warnings
```

### Benchmarks
```bash
just bench                   # Run all micro-benchmarks (divan); CI never runs benches
cargo bench --bench crypto   # Specific benchmark (crypto, disseminator, network, shredder)
```

### Simulations
Configure via constants in `src/bin/simulations/main.rs`:
```bash
./download_data.sh           # Required: download ping dataset first
RUST_LOG="simulations=debug" cargo run --release --bin=simulations
```

## Architecture Overview

### Core Components

The consensus protocol is built around three main abstractions that work together:

1. **Alpenglow** (`src/consensus.rs`) - The main consensus coordinator
   - Orchestrates all consensus activity via async message loops
   - Manages the lifecycle of block production, voting, and finalization
   - Connects `All2All`, `Disseminator`, `Blockstore`, `Pool`, and `Votor` components
   - **Validator** (`src/validator.rs`) wraps an `Alpenglow` consensus instance
     together with an `ExecutionEngine` to form a full node.

2. **Block Flow Pipeline**
   ```
   Leader produces block → Shredder → Disseminator → Network
                                                         ↓
   Network → Repair (if needed) → Blockstore → Pool → Votor → All2All → Certificates
   ```

3. **Key State Machines**
   - `Blockstore` (`src/consensus/blockstore.rs`) - Stores shreds and reconstructs blocks
   - `Pool` (`src/consensus/pool.rs`) - Tracks votes and certificates, manages finalization
   - `Votor` (`src/consensus/votor.rs`) - Main voting state machine, produces votes based on events

### Network Architecture

The system uses multiple independent network channels:

- **All2All** (`src/all2all/`) - Broadcasts votes and certificates to all validators
- **Disseminator** (`src/disseminator/`) - Disseminates block shreds (Rotor or Turbine)
- **Repair** (`src/repair.rs`) - Point-to-point repair requests/responses
- **Transaction Network** - Receives incoming transactions

Each network type is abstracted via traits (`All2All`, `Disseminator`, `Network`) with multiple implementations (UDP, TCP, simulated).

### Shredding System

Blocks are split into **slices**, and each slice is erasure-coded into **shreds**:

```
Block → Slices (fixed size chunks)
Each Slice → n data shreds + (k-n) coding shreds = k total shreds
```

- **Shredder** (`src/shredder.rs`) - Encodes slices into shreds with Reed-Solomon erasure coding
- **Merkle Trees** - Double-Merkle structure: Block Merkle tree over slice roots, each slice has its own Merkle tree over shreds
- **Shred** - Single UDP-sized packet with slice header, Merkle proof, and leader signature

Shredder implementations:
- `RegularShredder` - Standard data + coding shreds
- `CodingOnlyShredder` - Only coding shreds (data-availability focused)
- `AontShredder` / `PetsShredder` - All-or-nothing transform variants

### Consensus Flow

1. **Block Production** (`src/consensus/block_producer.rs`)
   - Leader for a slot produces a block
   - Shreds the block and sends via disseminator
   - Timing constraints: `DELTA_BLOCK` (400ms), `DELTA_FIRST_SLICE` (10ms)

2. **Block Reception & Reconstruction**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qkniep/alpenglow](https://github.com/qkniep/alpenglow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
