---
trigger: always_on
description: This guide provides comprehensive instructions for AI agents working on the **BNB Chain fork** of the Reth codebase. This is a production-grade execution layer client specifically adapted for **BNB Smart Chain (BSC)**.
---

# BNB Chain Reth (bnb-chain/reth) — Development Guide

This guide provides comprehensive instructions for AI agents working on the **BNB Chain fork** of the Reth codebase. This is a production-grade execution layer client specifically adapted for **BNB Smart Chain (BSC)**.

**Repository**: `bnb-chain/reth` (https://github.com/bnb-chain/reth)
**Upstream**: Fork of `paradigmxyz/reth` (v1.7.0 baseline)
**Main branch**: `main`

---

## Relationship: reth-bsc and reth

`bnb-chain/reth` (this repo) is the **core library layer** — a fork of upstream `paradigmxyz/reth` with BSC-specific modifications (Parlia consensus, TrieDB, system transactions, custom RPC, etc.).

`bnb-chain/reth-bsc` is the **application layer** that depends on this repo. It is the actual BSC node binary that imports crates from `bnb-chain/reth` as git dependencies. Issues reported in `reth-bsc` (e.g., RPC returning wrong data during staged sync) often have root causes in this repo's provider/storage layer.

When fixing issues from `reth-bsc`:
1. Trace the call path from RPC through the provider layer in this repo
2. Apply fixes here in `bnb-chain/reth`
3. The `reth-bsc` project will pick up fixes via git dependency updates

---

## BSC Fork Summary

This fork adapts the Ethereum-focused Reth client for BNB Smart Chain. Key differences from upstream:

| Aspect | Upstream Reth | BSC Fork |
|--------|--------------|----------|
| Consensus | PoS (Beacon Chain) | **Parlia** (Validator-based PoSA) |
| Finalized Block | Beacon Chain determined | Validator snapshot based |
| State Backend | MDBX only | **MDBX + TrieDB** (dual backend) |
| System Txs | Not supported | Supported (`gas_limit == u64::MAX/2`) |
| P2P Peers | Standard | **Proxied peer** support |
| Custom RPC | Standard Ethereum | `eth_getFinalizedBlock`, `eth_getFinalizedHeader` |
| Hardforks | Ethereum only | Ethereum + BSC-specific (Ramanujan → Pascal) |

### BSC-Specific Features

1. **Parlia Consensus Engine**: BSC's Proof-of-Staked-Authority (PoSA) consensus replacing Ethereum's PoS. Implementation in `examples/bsc-p2p/src/block_import/parlia.rs`. Canonical head: highest block number wins; for equal heights, lower hash wins.

2. **TrieDB State Storage Backend**: Alternative state database alongside MDBX. Configured via `StateDbConfig` in `crates/config/src/config.rs`. Supports genesis init, staged sync, live sync, and io-uring. Dependencies from `bnb-chain/reth-bsc-triedb`.

3. **BSC System Transactions**: Special transactions identified by `gas_limit == u64::MAX/2 && caller == beneficiary`. Exempted from block gas limit validation. Handled across all RPC tracers via `handle_bsc_system_transaction()` in `crates/rpc/rpc/src/debug.rs`.

4. **Custom RPC Methods**: `eth_getFinalizedBlock(verified_validator_num)` and `eth_getFinalizedHeader(verified_validator_num)` in `crates/rpc/rpc-eth-api/src/core.rs` and `helpers/block.rs`.

5. **Proxied Peer Support**: Enhanced P2P networking with proxy peer definitions in `crates/net/network/` and `crates/net/eth-wire-types/`.

6. **Validator Block Snapshot Table**: BSC validator support integrated into the engine with a dedicated database table.

7. **Blob Fee Validation**: Enhanced blob storage with extended time support and fee checks for EIP-4844 compatibility on BSC.

8. **BSC Hardforks**: Ramanujan, Niels, MirrorSync, Bruno, Euler, Nano, Moran, Gibbs, Planck, Luban, Plato, Hertz, HertzFix, Kepler, Feynman, FeynmanFix, Haber, HaberFix, Bohr, Pascal, Prague — defined in `examples/bsc-p2p/src/chainspec.rs`.

### Key BSC Code Locations

- `examples/bsc-p2p/` — Complete BSC P2P node example (Parlia consensus, chainspec, genesis)
- `crates/config/src/config.rs` — `StateDbConfig` for TrieDB/MDBX selection
- `crates/rpc/rpc/src/debug.rs` — BSC system transaction handling in tracers
- `crates/rpc/rpc-eth-api/src/core.rs` — Custom `eth_getFinalizedBlock` / `eth_getFinalizedHeader`
- `crates/rpc/rpc-eth-api/src/helpers/block.rs` — Finalized block logic with validator count
- `crates/transaction-pool/src/validate/eth.rs` — Blob fee and EIP-1559 validation for BSC
- `crates/net/network/` — Proxied peer support
- `crates/storage/provider/` — TrieDB integration in state provider

### Custom Dependencies

```toml
# In Cargo.toml — BSC TrieDB support
rust-eth-triedb = { git = "https://github.com/bnb-chain/reth-bsc-triedb.git", tag = "v0.0.1" }
rust-eth-triedb-common = { ... }
rust-eth-triedb-state-trie = { ... }
```

---

## Project Overview

Reth is a high-performance Ethereum execution client written in Rust, focusing on modularity, performance, and contributor-friendliness. The codebase is organized into well-defined crates with clear boundaries and responsibilities.

## Architecture Overview

### Core Components

1. **Consensus (`crates/consensus/`)**: Validates blocks according to consensus rules (Parlia for BSC)
2. **Storage (`crates/storage/`)**: Hybrid database using MDBX + static files + TrieDB (BSC) for optimal performance
3. **Networking (`crates/net/`)**: P2P networking stack with discovery, sync, transaction propagation, and proxied peer support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bnb-chain/reth](https://github.com/bnb-chain/reth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
