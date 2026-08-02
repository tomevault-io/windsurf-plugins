---
trigger: always_on
description: Dolos is a lightweight Cardano node designed specifically for keeping an updated copy of the ledger and responding to queries from trusted clients while requiring minimal resources compared to a full node. It serves as a data provider rather than a consensus-validating node, focusing on efficiency and compatibility with existing Cardano ecosystem tools.
---

# Dolos: Cardano Data Node - Rust Crate Organization

## Project Overview

Dolos is a lightweight Cardano node designed specifically for keeping an updated copy of the ledger and responding to queries from trusted clients while requiring minimal resources compared to a full node. It serves as a data provider rather than a consensus-validating node, focusing on efficiency and compatibility with existing Cardano ecosystem tools.

## Storage Concepts

Dolos uses four distinct storage backends, each serving a specific purpose:

### StateStore
- **Purpose**: Current ledger state (the "world view")
- **Contents**: UTxO set, entity state, chain cursor position
- **Traits**: `StateStore` (reads) + `StateWriter` (batched writes)
- **Database**: `<storage.path>/state`

### ArchiveStore
- **Purpose**: Historical block storage with temporal indexing
- **Contents**: Raw blocks indexed by slot, entity logs keyed by `LogKey` (slot + entity key)
- **Traits**: `ArchiveStore` (reads) + `ArchiveWriter` (batched writes)
- **Database**: `<storage.path>/chain`

### WalStore (Write-Ahead Log)
- **Purpose**: Crash recovery and rollback support
- **Contents**: Log entries with block data, entity deltas, and input UTxOs
- **Traits**: `WalStore`
- **Database**: `<storage.path>/wal`

### IndexStore
- **Purpose**: Cross-cutting indexes for fast lookups
- **Contents**: Two types of indexes:
  - **UTxO Filter Indexes**: Current state queries (by address, payment, stake, policy, asset)
  - **Archive Indexes**: Historical queries (by block hash, tx hash, slots with address/asset/etc.)
- **Traits**: `IndexStore` (reads) + `IndexWriter` (batched writes)
- **Database**: `<storage.path>/index` (isolated from other stores)
- **Design Note**: Returns primitive values (slots, UTxO refs) not block data. Use `QueryHelpers` to join with archive for full data.

### Database File Organization

```
<storage.path>/
├── wal      # Write-Ahead Log database
├── state    # Ledger state database
├── chain    # Archive/block storage database
└── index    # Consolidated index database
```

Each database is a separate Redb or Fjall file with independent configuration for cache size and durability, depending on the chosen storage backend.

## Crate Architecture

The project follows a modular workspace architecture with clear separation of concerns and trait-based extensibility.

### Core Crates

#### `dolos` (Main Binary)
- **Purpose**: Main application binary and CLI interface
- **Role**: Application layer that orchestrates all services
- **Key Modules**:
  - `sync`: Chain synchronization from upstream nodes
  - `serve`: gRPC (UTxO RPC) and Ouroboros network services
  - `relay`: Upstream relay connection handling
  - `mempool`: Transaction mempool implementation
  - `facade`: High-level domain operations (extends `dolos-core` facade)
- **CLI Commands**: daemon, sync, serve, bootstrap (relay/mithril/snapshot), data, doctor
- **Features**: Configurable service compilation (grpc, minibf, trp, mithril, utils)

#### `dolos-core` (Foundation)
- **Purpose**: Core traits, types, and abstractions common to all Dolos components
- **Key Modules**:
  - `state`: `StateStore` and `StateWriter` traits, entity system
  - `archive`: `ArchiveStore` and `ArchiveWriter` traits, `SlotTags` for indexing metadata
  - `indexes`: `IndexStore` and `IndexWriter` traits for cross-cutting indexes
  - `wal`: `WalStore` trait for write-ahead logging
  - `batch`: `WorkBatch`, `WorkBlock`, `WorkDeltas` for batch processing pipeline
  - `facade`: High-level operations (`execute_batch`, `roll_forward`, `import_blocks`)
  - `query`: `QueryHelpers` trait and `SparseBlockIter` for joining indexes with archive
  - `Domain`: Central trait tying all storage backends together
  - `ChainLogic`: Trait for blockchain-specific processing logic
  - `mempool`: Transaction mempool interface
- **Role**: Foundation layer providing the architecture that other crates implement

#### `dolos-cardano` (Blockchain Logic)
- **Purpose**: Cardano-specific implementation of the core traits
- **Components**:
  - `CardanoLogic`: Implementation of `ChainLogic` for Cardano
  - `CardanoEntity` / `CardanoDelta`: Entity-delta implementations
  - Block processing, validation, and era handling
  - Genesis configuration management and bootstrap
  - Reward distribution processing
  - UTxO set delta computation
- **Dependencies**: `dolos-core`, Pallas library for Cardano protocol support

#### `dolos-redb3` (Storage Backend)
- **Purpose**: Storage backend implementation using the Redb v3 embedded database
- **Components**:
  - `state`: `StateStore` implementation with UTxO storage and entity tables
  - `archive`: `ArchiveStore` implementation with block and log storage
  - `wal`: `WalStore` implementation for crash recovery
  - `indexes`: `IndexStore` implementation (isolated database) with:
    - UTxO filter indexes (by address, payment, stake, policy, asset)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [txpipe/dolos](https://github.com/txpipe/dolos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
