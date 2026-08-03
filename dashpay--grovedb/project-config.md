---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GroveDB is a hierarchical authenticated data structure database - essentially a "grove" (tree of trees) built on Merkle AVL trees. It provides efficient secondary index queries, cryptographic proofs, and is optimized for blockchain applications like Dash Platform.

## Development Commands

```bash
# Build the project
cargo build

# Run all tests
cargo test

# Run a specific test
cargo test test_name

# Run tests for a specific crate
cargo test -p grovedb
cargo test -p merk
cargo test -p storage

# Run with verbose output
cargo test -- --nocapture

# Build with all features
cargo build --features full,estimated_costs

# Run benchmarks
cargo bench

# Format code
cargo fmt

# Run linter
cargo clippy -- -D warnings

# Build documentation
cargo doc --open

# Run tests with specific features
cargo test --features full,verify
```

## Deep Architecture Understanding

### System Layers

1. **GroveDB Core** (`grovedb/src/`)
   - Orchestrates multiple Merk trees into a hierarchical structure
   - Each tree element can contain another Merk tree, creating a "grove"
   - Manages references between elements across trees
   - Handles batch operations atomically across multiple subtrees
   - Generates composite proofs spanning multiple trees

2. **Merk Layer** (`merk/src/`)
   - Merkle AVL tree implementation with self-balancing
   - Uses a unique node structure where intermediary nodes store key-value pairs
   - Supports lazy loading via the Link system (Reference/Modified/Uncommitted/Loaded)
   - Implements chunk-based restoration for large trees
   - Cost-aware operations with predefined costs for specialized nodes

3. **Storage Layer** (`storage/src/`)
   - Abstracts RocksDB with prefixed storage for subtree isolation
   - Uses Blake3 hashing to generate 32-byte prefixes from paths
   - Supports optimistic transactions via OptimisticTransactionDB
   - Four storage types: main data, auxiliary, roots, metadata
   - Batch operations minimize disk I/O

### Critical Design Patterns

#### Element System
```rust
// 8 element types with specific use cases:
Element::Item           // Basic key-value storage
Element::Reference      // Links between elements (7 reference types)
Element::Tree          // Container for subtrees
Element::SumItem       // Contributes to parent sum
Element::SumTree       // Maintains sum of descendants
Element::BigSumTree    // 256-bit sums
Element::CountTree     // Element counting
Element::CountSumTree  // Combined functionality
```

#### Reference Types
- **AbsolutePathReference**: Direct path from root
- **UpstreamRootHeightReference**: Navigate up N levels, then follow path
- **UpstreamFromElementHeightReference**: Relative to current element
- **CousinReference**: Same tree level, different branch
- **SiblingReference**: Same parent tree
- **UpstreamRootHeightWithParentPathAddition**: Complex navigation
- **UtilityReference**: System-level references

#### Cost Tracking
Every operation accumulates costs:
- `seek_count`: Database seeks
- `storage_loaded_bytes`: Data read from disk
- `storage_cost`: Added/replaced/removed bytes
- `hash_node_calls`: Cryptographic operations

### Key Implementation Details

#### Proof System
- Layer-by-layer proof generation from root to target
- Stack-based proof verification using virtual machine operations
- Supports absence proofs for non-existing keys
- Optimizes proof size by excluding unnecessary data

#### Query System (PathQuery)
```rust
PathQuery {
    path: Vec<Vec<u8>>,     // Starting location
    query: SizedQuery {
        query: Query {
            items: Vec<QueryItem>,  // What to select
            default_subquery_branch,
            conditional_subquery_branches,
            left_to_right: bool,
            add_parent_tree_on_subquery: bool,  // v2: Include parent tree in results
        },
        limit: Option<u16>,
        offset: Option<u16>,
    }
}
```

**New in v2**: The `add_parent_tree_on_subquery` flag allows including parent tree elements (like CountTree or SumTree) in query results when performing subqueries. This is useful when you need both aggregate values and individual elements.

#### Batch Operations
- Two-phase processing: validation then application
- TreeCache for deferred root hash propagation
- Atomic operations across multiple subtrees
- Support for transient operations

### Important Files and Their Roles

#### Core GroveDB
- `grovedb/src/grove_db.rs`: Main struct and public API
- `grovedb/src/operations/insert/mod.rs`: Insert logic with element validation
- `grovedb/src/operations/delete/mod.rs`: Delete operations including delete_up_tree
- `grovedb/src/operations/proof/generate.rs`: Multi-tree proof generation
- `grovedb/src/batch/mod.rs`: Batch operation processing
- `grovedb/src/reference_path/mod.rs`: Reference resolution logic

#### Merk Implementation
- `merk/src/tree/mod.rs`: AVL tree core with balancing
- `merk/src/tree/walk/mod.rs`: Walker pattern for lazy loading
- `merk/src/tree/ops.rs`: Tree operations (put, delete)
- `merk/src/proofs/query/mod.rs`: Query execution and proof generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dashpay/grovedb](https://github.com/dashpay/grovedb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
