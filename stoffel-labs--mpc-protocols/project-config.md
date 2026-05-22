---
trigger: always_on
description: This file provides guidance to Claude Code when working with the mpc-protocols repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the mpc-protocols repository.

## Repository Overview

`mpc-protocols` is a fully-asynchronous, Byzantine fault-tolerant collection of MPC (Multi-Party Computation) protocols designed for the Stoffel framework. It provides a modular architecture where multiple MPC protocol families share common networking, preprocessing, and arithmetic components.

**Primary protocol:** HoneyBadgerMPC
**Primary consumers:** Rust SDK, Python SDK, TypeScript SDK, StoffelVM
**Networking dependency:** `stoffelnet` (stoffel-networking)

## Development Commands

```bash
# Build
cargo build
cargo build --release

# Run all tests
cargo test

# Run specific test
cargo test test_mul

# Run tests with logging
RUST_LOG=info cargo test -- --nocapture

# Format and lint
cargo fmt
cargo clippy

# Generate documentation
cargo doc --open
```

## Repository Structure

```
mpc-protocols/
├── Cargo.toml              # Workspace definition
├── README.md
├── CLAUDE.md
├── mpc/                    # Core MPC protocol crate
│   ├── Cargo.toml
│   ├── src/
│   │   ├── lib.rs
│   │   ├── common/         # Shared components
│   │   │   ├── rbc/        # Reliable Broadcast
│   │   │   ├── share/      # Shamir secret sharing
│   │   │   └── types/      # Fixed-point, integer types
│   │   ├── honeybadger/    # HoneyBadgerMPC implementation
│   │   │   ├── mod.rs
│   │   │   ├── batch_recon/        # Batch reconstruction (Fig.2)
│   │   │   ├── double_share/       # Double share generation
│   │   │   ├── fpdiv/              # Fixed-point division
│   │   │   ├── fpmul/              # Fixed-point multiplication
│   │   │   ├── input/              # Distributed input via RBC
│   │   │   ├── mul/                # Secure Beaver multiplication
│   │   │   ├── output/             # Distributed output via RBC
│   │   │   ├── preprocessing.rs    # Preprocessing store
│   │   │   ├── ran_dou_sha/        # Random double sharing (Fig.3)
│   │   │   ├── robust_interpolate/ # RS interpolation (Fig.1)
│   │   │   ├── share_gen/          # Random share generation
│   │   │   └── triple_gen/         # Beaver triple generation
│   │   └── ffi/            # C FFI bindings for SDKs
│   │       └── c_bindings/
│   └── tests/              # Integration tests
└── network/                # Network trait implementations
    ├── Cargo.toml
    └── src/
        ├── lib.rs
        ├── fake_network.rs     # Testing network
        └── bad_fake_network.rs # Adversarial network for testing
```

## Architecture

### Core Components

| Component | Purpose |
|-----------|---------|
| `HoneyBadgerMPCNode<F, R>` | Main MPC node orchestrating all protocols |
| `Network` trait | Abstraction for message passing (from stoffelnet) |
| `RBC` trait | Reliable Broadcast Channel abstraction |
| `SecretSharingScheme<F>` | Shamir secret sharing over field F |

### Protocol Implementation Status

| Category | Protocol | Status |
|----------|----------|--------|
| Preprocessing | Shamir Random Share (RanSha) | Implemented |
| Preprocessing | Double Sharing / RanDouSha | Implemented |
| Preprocessing | Beaver Triple Generation | Implemented |
| Preprocessing | PRandBit / PRandInt | Implemented |
| Arithmetic | Secure Beaver Multiplication | Implemented |
| Arithmetic | Fixed-Point Mul / Div / Trunc | Implemented |
| I/O | Distributed Input/Output (RBC) | Implemented |
| Reconstruction | Robust Interpolation | Implemented |
| Reconstruction | Batch Reconstruction | Implemented |

### Message Routing

Messages are routed using compact `SessionId` fields:
- Protocol type identifier
- Instance ID
- Operation-specific indices

### MPC Configuration Requirements

| Parameter | Constraint |
|-----------|------------|
| `n_parties` | >= 5 for batch reconstruction benefits |
| `threshold` | n >= 3t + 1 (HoneyBadger requirement) |

## Key Files

### `mpc/src/honeybadger/mod.rs`
Main HoneyBadgerMPC node implementation:
- `HoneyBadgerMPCNode<F, R>` struct
- Protocol orchestration
- Message processing loop
- Preprocessing coordination

### `mpc/src/common/rbc/`
Reliable Broadcast Channel:
- `rbc.rs` - RBC trait and AVID implementation
- `rbc_store.rs` - Message storage
- Used for input/output protocols

### `mpc/src/honeybadger/mul/multiplication.rs`
Secure multiplication using Beaver triples:
- Core arithmetic operation
- Depends on preprocessing (triples)

### `mpc/src/honeybadger/preprocessing.rs`
Preprocessing store management:
- Random shares, double shares
- Beaver triples
- PRandBit/PRandInt outputs

### `mpc/src/ffi/c_bindings/`
C FFI exports for language SDK bindings:
- Python SDK uses these via ctypes
- TypeScript SDK uses via WASM bridge

## API Contracts

### With stoffel-networking

Uses `Network` trait from `stoffelnet`:
```rust
use stoffelnet::Network;

// Network methods used:
network.send(party_id, &data).await?;
network.broadcast(&data).await?;
let (sender, msg) = network.receive().await?;
```

### With StoffelVM

The VM calls MPC operations via the engine:
```rust
// Preprocessing
node.run_preprocessing(network, &mut rng).await?;

// Secure operations
node.mul(x_shares, y_shares, network).await?;
node.input(client_value, network).await?;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stoffel-Labs/mpc-protocols](https://github.com/Stoffel-Labs/mpc-protocols) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
