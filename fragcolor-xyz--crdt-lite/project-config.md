---
trigger: always_on
description: A lightweight, column-based CRDT (Conflict-free Replicated Data Type) implementation with both Rust and C++ versions.
---

# crdt-lite

A lightweight, column-based CRDT (Conflict-free Replicated Data Type) implementation with both Rust and C++ versions.

## Project Overview

This library provides generic CRDTs for building distributed, eventually-consistent data structures. The project includes:

1. **Column-based CRDT** (`src/lib.rs`, `crdt.hpp`) - Generic key-value store with column-level conflict resolution
2. **Text CRDT** (`text_crdt.hpp`) - Line-based collaborative text editor with fractional positioning

Both implementations support last-write-wins (LWW) semantics, tombstone-based deletion, logical clocks for causality tracking, and customizable merge rules.

## Architecture

### Dual Implementation Strategy

- **Rust** (`src/lib.rs`): Primary implementation with strong type safety, memory safety guarantees
- **C++** (`crdt.hpp`, `text_crdt.hpp`): Header-only template library for high-performance, zero-cost abstractions

Both implementations maintain API compatibility and use the same core algorithms.

### Core Concepts

#### 1. Column-Based Design

Records are stored as maps of columns (field names) to values. Each column has independent version tracking:

```rust
// Rust
pub struct Record<V> {
  pub fields: HashMap<ColumnKey, V>,
  pub column_versions: HashMap<ColumnKey, ColumnVersion>,
  pub lowest_local_db_version: u64,
  pub highest_local_db_version: u64,
}
```

```cpp
// C++
template <typename V> struct Record {
  CrdtMap<CrdtKey, V> fields;
  CrdtMap<CrdtKey, ColumnVersion> column_versions;
  uint64_t lowest_local_db_version = UINT64_MAX;
  uint64_t highest_local_db_version = 0;
};
```

**Why column-based?**
- Fine-grained conflict resolution (conflicts resolved per-field, not per-record)
- Efficient sync (only changed columns need syncing)
- Natural fit for structured data (forms, database records)

#### 2. Version Tracking

Each change is tracked with both column-level and global version information:

```rust
pub struct ColumnVersion {
  pub col_version: u64,      // Per-column version (increments on each edit)
  pub db_version: u64,       // Global logical clock at creation time
  pub node_id: NodeId,       // Which node made this change
  pub local_db_version: u64, // Local clock when applied (for sync)
}
```

**Conflict Resolution Priority:**
1. Column version (higher wins)
2. DB version (higher wins)
3. Node ID (higher wins as deterministic tie-breaker)

#### 3. Logical Clocks

Maintains causality using Lamport-style logical clocks:

```rust
impl LogicalClock {
  pub fn tick(&mut self) -> u64 {
    self.time += 1;
    self.time
  }

  pub fn update(&mut self, received_time: u64) -> u64 {
    self.time = self.time.max(received_time);
    self.time += 1;
    self.time
  }
}
```

**Important:** Always update clock on merge, even for rejected changes (prevents clock drift, maintains causal consistency).

#### 4. Tombstone-Based Deletion

Deleted records are marked with tombstones rather than immediately removed:

```rust
pub struct TombstoneInfo {
  pub db_version: u64,
  pub node_id: NodeId,
  pub local_db_version: u64,
}
```

**Tombstone Management:**
- Tombstones accumulate indefinitely unless compacted
- **CRITICAL:** Only compact after ALL nodes acknowledge a version
- Early compaction causes deleted records to reappear (zombie records)
- See `compact_tombstones()` docs in src/lib.rs:1002-1024

**DoS Protection:**
- Tombstones can grow unbounded → memory exhaustion
- Implement application-level rate limiting
- Track acknowledged versions across all nodes
- Call `compact_tombstones(min_acknowledged_version)` periodically

#### 5. Parent-Child Hierarchies

CRDTs can have parent-child relationships for layered state:

```rust
pub struct CRDT<K, V> {
  parent: Option<Arc<CRDT<K, V>>>,
  base_version: u64,  // Parent's version at child creation
  // ...
}
```

**Use cases:**
- Temporary overlays (preview changes without committing)
- Transaction isolation
- Branching workflows

**Operations:**
- `revert()`: Generate inverse changes to undo child modifications
- `diff()`: Compute changes between two CRDTs
- Child sees parent data but maintains separate modifications

#### 6. Sorted Keys (Optional Feature)

The Rust implementation supports an optional `sorted-keys` feature that replaces the internal HashMap with BTreeMap for sorted key storage and range queries.

**Enable in Cargo.toml:**
```toml
[dependencies]
crdt-lite = { version = "0.7", features = ["sorted-keys"] }
```

**Use Cases:**
- **Composite Keys:** Use lexicographic ordering for hierarchical keys like `"session-{uuid}-{index}"` or `"user-{id}-{timestamp}"`
- **Range Queries:** Efficiently query all records within a key range
- **Ordered Iteration:** Iterate over records in sorted key order

**Performance:**
- **HashMap (default):** O(1) lookups, unordered iteration
- **BTreeMap (sorted-keys):** O(log n) lookups, ordered iteration, range queries
- **Verdict:** Log n overhead is negligible compared to network sync and persistence I/O

**Range Query API:**
```rust
// Get all records for a specific session
for (key, record) in crdt.range("session-abc-".."session-abd-") {
    println!("Found: {:?}", record);
}

// Get all records with keys >= "user-100"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fragcolor-xyz/crdt-lite](https://github.com/fragcolor-xyz/crdt-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
