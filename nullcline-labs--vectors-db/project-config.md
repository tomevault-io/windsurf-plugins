---
trigger: always_on
description: In-memory vector database written in Rust. Supports HNSW approximate nearest neighbor search, BM25 full-text search, and hybrid retrieval with RRF/linear fusion. Designed for RAG applications and semantic search.
---

# CLAUDE.md — vectors.db

## Project Overview

In-memory vector database written in Rust. Supports HNSW approximate nearest neighbor search, BM25 full-text search, and hybrid retrieval with RRF/linear fusion. Designed for RAG applications and semantic search.

## Build & Test Commands

```bash
cargo build --release          # Build optimized binary
cargo test                     # Run all tests (6 unit + 45 integration)
cargo clippy -- -D warnings    # Lint (zero warnings policy)
cargo fmt --check              # Format check
cargo bench                    # Run all 4 benchmarks (GloVe-25, SIFT-128, GloVe-100, BM25 NFCorpus)
cargo bench --bench ann_glove25   # Single benchmark
```

## Run

```bash
# Standalone
cargo run --release -- --port 3030 --data-dir ./data

# With auth
VECTORS_DB_API_KEY=secret cargo run --release

# With RBAC
VECTORS_DB_API_KEYS='[{"key":"admin-key","role":"admin"},{"key":"read-key","role":"read"}]' cargo run --release

# With TLS
cargo run --release -- --tls-cert cert.pem --tls-key key.pem

# With memory limit and auto-snapshots
cargo run --release -- --max-memory-mb 4096 --snapshot-interval 300

# Cluster mode (Raft)
cargo run --release -- --node-id 1 --peers "2=host2:3030,3=host3:3030"
```

## Architecture

```
src/
├── main.rs                 # CLI args, WAL replay, Raft init, graceful shutdown
├── lib.rs                  # Module declarations
├── config.rs               # Constants (limits, defaults, BM25/HNSW params)
├── document.rs             # Document struct, MetadataValue enum
├── api/
│   ├── mod.rs              # Axum router, middleware stack (auth, metrics, request-id, rate-limit, timeout, body-limit)
│   ├── handlers.rs         # All HTTP handlers, AppState struct
│   ├── models.rs           # Request/response DTOs
│   ├── errors.rs           # ApiError enum → HTTP status codes
│   ├── rbac.rs             # Role-based access control (Read/Write/Admin)
│   └── metrics.rs          # Prometheus metrics (http_requests_total, etc.)
├── hnsw/
│   ├── graph.rs            # HnswIndex struct, HnswConfig, layer management
│   ├── search.rs           # search_layer (generic filter_fn), knn_search, knn_search_filtered
│   ├── insert.rs           # HNSW insertion with layer selection
│   ├── distance.rs         # Cosine, Euclidean, DotProduct (f32 + quantized u8)
│   └── visited.rs          # VisitedSet for graph traversal
├── bm25/
│   ├── inverted_index.rs   # InvertedIndex with postings lists
│   ├── scorer.rs           # BM25 Okapi scoring (k1=1.2, b=0.75)
│   └── tokenizer.rs        # Whitespace tokenizer (no stemming/stopwords)
├── quantization/
│   └── scalar.rs           # f32→u8 scalar quantization with min/scale per vector
├── search/
│   ├── types.rs            # ScoredDocument
│   ├── filter.rs           # matches_filter() for metadata predicates
│   └── hybrid.rs           # rrf_fusion(), linear_fusion()
├── storage/
│   ├── collection.rs       # Collection, CollectionData, Database structs
│   ├── wal.rs              # Write-Ahead Log (CRC32 + fsync)
│   └── persistence.rs      # Bincode snapshot save/load, atomic writes
└── cluster/
    ├── types.rs            # openraft TypeConfig, LogEntry enum, NodeId
    ├── store.rs            # InMemoryLogStore, StateMachineStore (applies to Database)
    ├── network.rs          # HTTP-based Raft RPC (reqwest)
    └── api.rs              # Raft internal routes (/raft/vote, /raft/append, etc.)
```

## Key Design Decisions

- **In-memory only**: All data lives in RAM. WAL + snapshots provide durability, not disk-based storage.
- **Scalar quantization**: f32→u8 reduces memory 4x. Asymmetric distance (query f32 vs stored u8) for search, f32 reranking for accuracy.
- **Pre-filtering in HNSW**: Filter predicate `Fn(u32) -> bool` applied during graph traversal (not post-retrieval). Filtered nodes still used for navigation.
- **Centralized ID mapping**: `uuid_to_internal: HashMap<Uuid, u32>` and `internal_to_uuid: Vec<Uuid>` shared by HNSW and BM25 indices.
- **parking_lot::RwLock**: Low-contention read-write locks. Collection-level granularity.
- **openraft 0.9 with storage-v2**: Split `RaftLogStorage` + `RaftStateMachine` traits. BTreeMap-based in-memory log store.

## API Routes

### Public (no auth)
- `GET /health` — `{"status":"ok","version":"0.1.0"}`
- `GET /metrics` — Prometheus text format

### Collections (auth required)
- `POST /collections` — Create: `{"name","dimension","m?","ef_construction?","ef_search?","distance_metric?"}`
- `GET /collections` — List all
- `DELETE /collections/:name` — Delete

### Documents (auth required)
- `POST /collections/:name/documents` — Insert: `{"text","embedding","metadata?","id?"}`
- `POST /collections/:name/documents/batch` — Batch insert (max 1000): `{"documents":[...]}`
- `GET /collections/:name/documents/:id` — Get by UUID
- `PUT /collections/:name/documents/:id` — Update: `{"text?","embedding?","metadata?"}`
- `DELETE /collections/:name/documents/:id` — Delete

### Search (auth required)
- `POST /collections/:name/search` — `{"query_text?","query_embedding?","k","offset?","min_similarity?","alpha?","fusion_method?","filter?"}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nullcline-labs/vectors.db](https://github.com/nullcline-labs/vectors.db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
