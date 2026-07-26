---
trigger: always_on
description: > Comprehensive atomic-level documentation for AI agents working with this codebase.
---

# Puffer Vector Database - Agent Reference

> Comprehensive atomic-level documentation for AI agents working with this codebase.

## 1. Project Overview

**Puffer** is a high-performance, single-node vector database written in Rust implementing:
- **IVF-Flat** (Inverted File with Flat quantization) indexing
- **K-means++** clustering for index construction
- **Memory-mapped** immutable segment files
- **LSM-style** compaction (L0 → L1)
- **REST HTTP API** via Axum

---

## 2. Directory Structure

```
puffer-mvp/
├── Cargo.toml                    # Workspace configuration
├── Cargo.lock                    # Dependency lock file
├── README.md                     # User documentation
├── BENCHMARKS.md                 # Performance benchmarks
├── Agent.md                      # This file
│
├── crates/
│   ├── core/                     # Vector math & types
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── types.rs          # VectorId, VectorRecord
│   │       ├── metric.rs         # Metric enum (L2, Cosine)
│   │       └── distance.rs       # SIMD distance functions
│   │
│   ├── storage/                  # Persistence layer
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── segment.rs        # Segment file format
│   │       ├── catalog.rs        # Collection management
│   │       ├── router.rs         # Segment routing index
│   │       └── error.rs          # StorageError
│   │
│   ├── index/                    # Indexing algorithms
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── kmeans.rs         # K-means++ clustering
│   │       └── search.rs         # IVF-Flat search
│   │
│   ├── query/                    # Query execution
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── engine.rs         # QueryEngine
│   │       ├── compaction.rs     # LSM compaction
│   │       └── error.rs          # QueryError
│   │
│   ├── api/                      # HTTP server
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── main.rs           # puffer-server binary
│   │       ├── routes.rs         # Router definition
│   │       ├── handlers.rs       # Request handlers
│   │       └── state.rs          # AppState
│   │
│   ├── cli/                      # CLI tools
│   │   └── src/
│   │       └── main.rs           # puffer-cli binary
│   │
│   └── bench/                    # Benchmarks
│       └── src/
│           ├── lib.rs
│           └── bin/
│               ├── recall_bench.rs
│               └── real_recall_bench.rs
│
├── scripts/
│   ├── download_glove.py         # Download ANN-benchmarks data
│   ├── plot_recall_latency.py    # Plotting scripts
│   └── plot_real_recall.py
│
└── data/                         # Runtime data (gitignored)
    ├── catalog.json
    └── {collection}/
        ├── meta.json
        ├── router_index.json
        └── {uuid}.seg
```

---

## 3. Crate Dependencies

### 3.1 Internal Dependencies
```
puffer-api     → puffer-query → puffer-index → puffer-core
                             → puffer-storage → puffer-core
puffer-cli     → puffer-core (via reqwest to API)
puffer-bench   → puffer-query, puffer-storage, puffer-core
```

### 3.2 Key External Dependencies

| Crate | Version | Purpose |
|-------|---------|---------|
| `tokio` | 1.35 | Async runtime |
| `axum` | 0.7 | HTTP framework |
| `serde` / `serde_json` | 1.0 | Serialization |
| `rayon` | 1.8 | Parallel iteration |
| `memmap2` | 0.9 | Memory-mapped files |
| `clap` | 4.4 | CLI parsing |
| `uuid` | 1.6 | Segment naming |
| `tracing` | 0.1 | Structured logging |
| `rand` | 0.8 | Random number generation |
| `ndarray` / `ndarray-npy` | - | NumPy file loading (bench) |

---

## 4. Core Data Types

### 4.1 VectorId (`crates/core/src/types.rs`)
```rust
pub struct VectorId(pub String);

impl VectorId {
    pub fn new(id: impl Into<String>) -> Self;
    pub fn as_str(&self) -> &str;
    pub fn to_bytes(&self) -> Vec<u8>;      // [len: u8][bytes...]
    pub fn from_bytes(data: &[u8]) -> Option<(Self, usize)>;
}
```
- Max length: 255 bytes (length-prefixed)
- Used as primary key for vectors

### 4.2 VectorRecord (`crates/core/src/types.rs`)
```rust
pub struct VectorRecord {
    pub id: VectorId,
    pub vector: Vec<f32>,
    pub payload: Option<serde_json::Value>,
}
```

### 4.3 Metric (`crates/core/src/metric.rs`)
```rust
pub enum Metric {
    L2,      // Euclidean (uses squared distance internally)
    Cosine,  // Angular (1 - cosine_similarity)
}

impl Metric {
    pub fn to_byte(&self) -> u8;        // L2=0, Cosine=1
    pub fn from_byte(b: u8) -> Option<Self>;
}
```

---

## 5. Distance Functions (`crates/core/src/distance.rs`)

### 5.1 Available Functions

| Function | Returns | Notes |
|----------|---------|-------|
| `l2_distance_squared(a, b)` | `f32` | **Fastest** - no sqrt |
| `l2_distance(a, b)` | `f32` | Full Euclidean |
| `dot_product(a, b)` | `f32` | Vector dot product |
| `l2_norm(v)` | `f32` | Vector magnitude |
| `cosine_similarity(a, b)` | `f32` | Range: [-1, 1] |
| `cosine_distance(a, b)` | `f32` | Range: [0, 2] |
| `cosine_distance_with_norms(a, b, norm_a, norm_b)` | `f32` | **Fast** - precomputed norms |
| `distance(a, b, metric)` | `f32` | Generic dispatch |
| `normalize(v)` | `()` | In-place normalization |

### 5.2 SIMD Optimization Pattern
```rust

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harishsg993010/open-puffer](https://github.com/harishsg993010/open-puffer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
