---
trigger: always_on
description: Atomic is a mathematically sound distributed version control system built in Rust. It uses **patch theory** to represent changes as composable, commutative operations on a directed graph, enabling conflict-free merges when changes are truly independent.
---

# AGENTS.md - Atomic Development Guide

## Project Overview

Atomic is a mathematically sound distributed version control system built in Rust. It uses **patch theory** to represent changes as composable, commutative operations on a directed graph, enabling conflict-free merges when changes are truly independent.

### Design Philosophy

1. **Mathematical Soundness**: Changes are algebraic operations with well-defined composition rules
2. **Content-Addressed**: All data is identified by cryptographic hashes (Blake3)
3. **Graph-Based**: Files are DAGs of vertices and edges, not linear sequences
4. **Views, Not Forks**: Views are filtered perspectives on the same graph, not divergent histories

## Architecture

### Crate Structure

```
atomic/
├── atomic-cli/           # CLI application
├── atomic-core/          # Core VCS engine
│   ├── types/            # Fundamental data types
│   └── pristine/         # Storage layer (redb)
├── atomic-config/        # Configuration management
├── atomic-identity/      # User identity & Ed25519 signing
└── atomic-repository/    # High-level repository operations
```

### Related Projects

- **atomic-remote-client** (`atomic-enterprise/atomic-remote`) - Clean-room HTTP client for remote operations
- **atomic-api** (`atomic-enterprise/atomic-api`) - Server-side HTTP API for remote operations

## Core Concepts

### 1. Repository Graph

Files are represented as directed acyclic graphs (DAGs). Nodes are opaque
byte ranges (hunks); edges define the ordering between them. The semantic
layer (CRDT) interprets the bytes as human-readable text.

```
  Graph layer (storage):

  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │ Hunk [0:5]   │────▶│ Hunk [5:6]   │────▶│ Hunk [0:5]   │
  │ (5 bytes)    │     │ (1 byte)     │     │ (5 bytes)    │
  └──────────────┘     └──────────────┘     └──────────────┘
  Node (change 1)      Node (change 1)      Node (change 2)

  Semantic layer (interpretation):

  The CRDT reads the hunks and translates them for display:
  [0:5] → "Hello"    [5:6] → " "    [0:5] → "World"
```

- **Inodes** (`Inode`): A stable identifier for a file. Survives renames.
  The `TREE` table maps `path → Inode` and `INODES` maps `Inode → Position`
  (the root node of that file's graph). The Inode *is* the file.
- **Nodes / Vertices** (`GraphNode`): Each node holds a chunk of content (a
  byte range within a change). The codebase uses "node" and "vertex"
  interchangeably — `GraphNode` is the struct name, "vertex" is the graph
  theory term used in traversal code (`AliveVertex`, `find_block`, etc.).
  To read a file, walk the graph and concatenate node content in edge order.
- **Edges** (`SerializedGraphEdge`): Define ordering between nodes. An edge
  from A to B means "A's content comes before B's." Edges carry flags
  (`BLOCK`, `FOLDER`, `PARENT`, `DELETED`) that indicate structure and state.

### 2. Changes (Patches)

Atomic transformations that add/remove vertices and edges:

```rust
// A change is identified by its content hash
let hash = Hash::of(change_content);

// Changes are registered to get a repository-local ID
let node_id = txn.register_change(&hash)?;
```

### 3. Views (Ambient Graph + View Filters)

**Critical Concept**: Views are **change-set filters** on a single canonical
GRAPH. All edges are always written to the global GRAPH immediately. A view
determines which subset of the graph is visible by tracking which changes
belong to it in `VIEW_CHANGES`.

| Aspect | Git Branches | Atomic Views |
|--------|--------------|--------------|
| Data Model | Pointer to commit | Ordered set of change references |
| Storage | Duplicates history | Single GRAPH, per-view change filters |
| "Merging" | 3-way merge | Insert change references (with dependency closure) |
| State | HEAD commit hash | Merkle hash of change sequence |
| Cleanup | Manual branch delete + GC | Delete VIEW_CHANGES entries; GC orphaned edges |
| Collaboration | Isolated snapshots | Filtered perspectives, real-time sharing |

#### View Scopes

```rust
pub enum ViewScope {
    /// Personal workspace. Changes visible through the view's filter.
    /// Deletion removes VIEW_CHANGES entries; orphaned edges cleaned by GC.
    Draft,  // feature, bug, service-auth, experiment

    /// Collaborative view. Visible to all. Deletion is restricted.
    Shared,    // dev, release, main
}
```

#### Parent Chains and View Filters

Every view has a parent (except the root). The parent relationship defines
the **filter chain** for graph traversal:

```
  main  (Shared, parent=None — the only true root)
    │
  release  (Shared, parent=main)
    │
  dev  (Shared, parent=release)
    │
    ├── service-auth  (Draft, parent=dev)
    │     ├── feature-login   (Draft, parent=service-auth)
    │     └── feature-logout  (Draft, parent=service-auth)
    │
    └── service-payments  (Draft, parent=dev)
```

A view's **effective filter** is the union of its own `VIEW_CHANGES` and
all ancestor views' `VIEW_CHANGES`, plus the transitive dependency closure:

```
feature-login filter = VIEW_CHANGES[feature-login]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomicdotdev/atomic](https://github.com/atomicdotdev/atomic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
