---
trigger: always_on
description: A client-side vector search library with a Rust/WASM core and a clean
---

# VecLite

## What this is
A client-side vector search library with a Rust/WASM core and a clean
TypeScript API. Designed to work where pure JS solutions break down —
10k, 50k, 100k+ vectors in the browser. No server. No API keys. Just search.

## Core positioning
"Client-side vector search that scales.
Rust/WASM powered, works where pure JS breaks down."

## Repository structure
```
veclite/
├── CLAUDE.md
├── DECISIONS.md
├── README.md
├── package.json
├── tsconfig.json
├── .github/
│   └── workflows/
│       └── ci.yml
├── rust/
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs
│       ├── index.rs
│       ├── similarity.rs
│       ├── filter.rs
│       └── types.rs
├── src/
│   ├── index.ts
│   ├── veclite.ts
│   ├── types.ts
│   ├── validator.ts
│   ├── utils.ts
│   ├── adapters/
│   │   ├── adapter.ts        ← StorageAdapter interface
│   │   ├── indexeddb.ts      ← default browser adapter
│   │   └── memory.ts         ← in-memory / testing
│   └── rag/                  ← veclite/rag sub-path entry
│       ├── index.ts          ← public API
│       ├── types.ts          ← VecLiteRAGConfig, RAGSearchResult, ProgressCallback
│       ├── pipeline.ts       ← VecLiteRAG class
│       └── chunker.ts        ← text chunking
├── tests/
├── bench/
└── docs/
```

## Architecture layers
Three distinct layers — never mix concerns across them:

1. TypeScript API layer — input validation, error handling, persistence, DX
2. WASM boundary — keep crossings minimal, always batch, use Float32Array
3. Rust/WASM core — pure computation only, no async, no JS concepts

### Golden rule
Rust does pure computation only.
All async, persistence, and validation lives in TypeScript.

## Decisions locked

### Algorithm
- v0.1: Flat index, brute force cosine similarity
- v0.2: SIMD-accelerated cosine similarity (core::arch::wasm32 f32x4, simd Cargo feature)
- v0.3: HNSW approximate nearest neighbour (hnsw crate, M=16/M0=32, opt-in via indexType: 'hnsw')

### Metadata filtering
- v0.1: Exact match only — { category: 'science' }
- v0.2: Operator predicates — $gte, $lte, $in, $ne; exact match fully backwards compatible
- Flat index: Pre-filter (filter candidates first, then rank by similarity)
- HNSW: Post-filter with oversample=10 (fetch topK×10 from graph, apply filter, return topK)
- Rationale: Pre-filter impossible in HNSW without graph knowledge

### Data types
- f32 only (not f64)
- Rationale: OpenAI embeddings are f32, halves memory, better SIMD compatibility

### Build toolchain
- wasm-pack for compilation
- wasm-bindgen for JS/Rust bindings

### Similarity metric
- v0.1: Cosine similarity only
- v0.2: Cosine similarity with SIMD acceleration
- v0.3: L2 (Euclidean) and dot product added; metric specified at construction time (Metric = 'cosine' | 'l2' | 'dot')
- Dot product in HNSW uses cosine distance in the graph (optimized for unit vectors); actual dot product recomputed at score time

### WASM loading
- Explicit init: await VecLite.init() before first use
- Rationale: Explicit is better than implicit for a library

### Bundle strategy
- External .wasm file (not inlined as base64)
- Rationale: Bundle size matters, developers can handle hosting

### Persistence — Storage Adapter pattern
- Pure generic key/value interface — no domain knowledge in the adapter
- Adapter does not know about vectors, metadata, or search
- VecLite owns all serialisation/deserialisation
- Ships with two adapters: IndexedDBAdapter (default) and MemoryAdapter (testing/in-memory)
- Community builds SQLite, AsyncStorage, localStorage etc.
- v0.2: Evaluate Repository pattern if partial loads or native querying become necessary

```typescript
// adapter.ts — the full interface, nothing more
interface StorageAdapter {
  get(key: string): Promise<string | null>
  set(key: string, value: string): Promise<void>
  delete(key: string): Promise<void>
  clear(): Promise<void>
}
```

Usage:
```typescript
// Default — IndexedDB
const db = new VecLite({ dimensions: 1536 })

// Custom adapter
const db = new VecLite({
  dimensions: 1536,
  storage: new MyCustomAdapter()
})

// In-memory only (no persistence)
const db = new VecLite({
  dimensions: 1536,
  storage: new MemoryAdapter()
})
```

### API style
- Class-based (VecLite), not functional
- Methods: upsert, search, delete, save, load
- Metadata: Record<string, string | number | boolean>

## API (v0.3)

```typescript
import { VecLite, IndexedDBAdapter, MemoryAdapter } from 'veclite'
import type { FilterOperator, FilterValue, Metric, IndexType } from 'veclite'

// Explicit init — required before first use
await VecLite.init()

// Default (IndexedDB, flat index, cosine metric)
const db = new VecLite({ dimensions: 1536 })

// HNSW index with L2 metric
const db = new VecLite({
  dimensions: 1536,
  indexType: 'hnsw',
  metric: 'l2',
  efConstruction: 200,    // HNSW build quality (default: 200)
})

// Custom storage adapter
const db = new VecLite({ dimensions: 1536, storage: new MyAdapter() })

// In-memory only
const db = new VecLite({ dimensions: 1536, storage: new MemoryAdapter() })

// Upsert
db.upsert([
  { id: 'doc1', vector: [...], metadata: { category: 'science', year: 2024 } }
])

// Search — exact match (v0.1, still works)
const results = db.search({
  vector: [...],
  topK: 5,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thealpha93/VecLite](https://github.com/thealpha93/VecLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
