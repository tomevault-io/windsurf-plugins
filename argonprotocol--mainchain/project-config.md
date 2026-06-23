---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Project Overview

Argon is an inflation-proof stablecoin backed by Bitcoin and the Ulixee data network. The codebase
is primarily written in Rust using the Substrate framework for the blockchain layer, with
TypeScript/Node.js bindings for client applications.

## Key Architecture Components

### Core Components

- **Mainchain** - The L1 blockchain built on Substrate that handles stabilization, consensus, and
  Bitcoin locking
- **Localchain** - The L2 personal blockchain layer for high-frequency transactions and
  micropayments
- **Notary** - Validates localchain transactions and submits notebooks to mainchain
- **Oracle** - Provides price data and Bitcoin tip information to the blockchain
- **Bitcoin Integration** - Handles Bitcoin locking, vaults, and miniscript for backing Argons

### Consensus Model

- **Slot Zero** - RandomX proof-of-work for bootstrapping
- **Registered Mining** - Miners bid for slots and are matched with best localchain votes using XOR
  closest algorithm

## Development Commands

### Building

```bash
# Build everything (includes sqlx setup)
cargo make build


# Build TypeScript/Node.js packages
yarn build                     # Build all JS packages
yarn tsc                       # TypeScript compilation only
```

### Testing

```bash
# Run Rust tests (starts minio automatically)
cargo make test

# Run specific Rust tests
cargo test --package <package-name>

# Run JavaScript tests
yarn test

# Run tests with nextest (faster)
cargo make nextest
```

### Linting and Formatting

```bash
# Rust
cargo make lint         # Run clippy with fixes
cargo make format       # Run zepter and rustfmt

# JavaScript/TypeScript
yarn lint               # Format all JS/TS files with prettier
```

### Database Setup

```bash
# Setup SQLx databases for localchain and notary
cargo make sqlx-setup

# Prepare SQLx queries
cargo make sqlx
```

### Running Local Testnet

```bash
# Start local test network (requires PostgreSQL and MinIO)
./scripts/local_testnet/start.sh

# Start MinIO for object storage
./scripts/docker_minio.sh
```

### Docker

```bash
# Build Docker images
cargo make docker
```

### Release Process

```bash
# Create a new release with changelog updates
cargo make release         # Auto-bump version
cargo make release v1.4.2  # Use an explicit version
cargo make release --major   # Major version bump
cargo make release --minor   # Minor version bump
cargo make release --patch   # Patch version bump
cargo make release --bump # automatic bump based on changes
```

## Development Dependencies

### Required Services

- **PostgreSQL 14+** - Running at localhost:5432 (for notary and tests)
- **MinIO** - Running at localhost:9000 (for notebook archival)
  - Start with: `./scripts/docker_minio.sh`

### Environment Variables

```bash
# Set mainchain URL (optional)
export ARGON_MAINCHAIN_URL=wss://rpc.argon.network  # For mainnet
export ARGON_MAINCHAIN_URL=wss://rpc.testnet.argonprotocol.org  # For testnet
```

## Workspace Structure

### Rust Workspaces

- `node/` - Blockchain node implementation with consensus and bitcoin UTXO tracking
- `runtime/` - Runtime logic defining state transitions (argon and canary variants)
- `pallets/` - Blockchain modules (bitcoin_locks, mining_slot, notebook, etc.)
- `primitives/` - Shared types and models across all components
- `localchain/` - L2 implementation with SQLite storage
- `notary/` - Notebook validation and submission service
- `bitcoin/` - Bitcoin integration CLI and vault management
- `client/` - Rust and Node.js client libraries
- `testing/` - Integration testing framework

### TypeScript/Node.js Packages

- `client/nodejs/` - TypeScript client for mainchain interactions
- `bitcoin/nodejs/` - TypeScript client for Bitcoin integration
- `localchain/` - Node.js bindings for localchain (using napi-rs)
- `testing/nodejs/` - JavaScript testing utilities

## Common Development Tasks

### Working with Pallets

Pallets define the blockchain's storage, transactions, and logic. Key pallets:

- `bitcoin_locks` - Lock/release Bitcoin to vault
- `mining_slot` - Mining slot registration and bidding
- `notebook` - Track notary notebooks and account changes
- `localchain_transfer` - Transfer between chains (Localchain ↔ Mainchain)

### Updating Chain Metadata

```bash
# After modifying runtime, update TypeScript definitions
cargo make update-metadata
```

### Running a Single Test

```bash
# Rust
cargo test -p <package-name> <test_name>

# JavaScript
cd client/nodejs && yarn test <test-file>
```

## Cross-Platform Considerations

### Localchain Bindings

- Node.js bindings use napi-rs
- iOS/Android bindings use uniffi
- Supports darwin (x64/arm64), linux (x64/arm64), windows (x64)

### Bitcoin Integration

The Bitcoin CLI requires careful handling of file paths and key management. Always use absolute
paths when working with Bitcoin-related commands.

## Network URLs

### Mainnet (Experimental)

- RPC: `wss://rpc.argon.network`
- Notary: `wss://notary1.argon.network`
- Bootnodes: `wss://bootnode0.argon.network`, `wss://bootnode1.argon.network`

### Testnet

- RPC: `wss://rpc.testnet.argonprotocol.org`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argonprotocol/mainchain](https://github.com/argonprotocol/mainchain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
