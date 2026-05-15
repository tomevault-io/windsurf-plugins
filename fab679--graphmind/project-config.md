---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Graphmind is a high-performance distributed graph database written in Rust with ~90% OpenCypher query support, Redis protocol (RESP) compatibility, multi-tenancy, vector search, NLQ, and graph algorithms. Currently at Phase 4 (High Availability Foundation), version v0.6.1.

## Build & Development Commands

```bash
# Build
cargo build                    # Debug build
cargo build --release          # Release build (optimized)

# Run tests (1842 unit tests)
cargo test                     # All tests
cargo test graph::node         # Specific module tests
cargo test -- --nocapture      # Tests with output

# Benchmarks (all in benches/)
cargo bench                                    # All benchmarks
cargo bench --bench graph_benchmarks           # Criterion micro-benchmarks (15 benches)
cargo bench --bench full_benchmark             # Full suite (ingestion, vector, traversal, algorithms)
cargo bench --bench vector_benchmark           # HNSW vector search
cargo bench --bench graphalytics_benchmark     # LDBC Graphalytics algorithms
cargo bench --bench mvcc_benchmark             # MVCC & arena allocation
cargo bench --bench late_materialization_bench  # Late materialization traversal
cargo bench --bench graph_optimization_benchmark # Metaheuristic optimization solvers
cargo bench --bench ldbc_benchmark             # LDBC SNB Interactive queries (needs data)
cargo bench --bench ldbc_bi_benchmark          # LDBC SNB BI queries (needs data)
cargo bench --bench finbench_benchmark         # LDBC FinBench queries (synthetic data)

# Run examples
cargo run --example banking_demo              # Banking fraud detection + NLQ
cargo run --example clinical_trials_demo      # Clinical trials + vector search
cargo run --example supply_chain_demo         # Supply chain + optimization
cargo run --example smart_manufacturing_demo  # Digital twin + scheduling
cargo run --example social_network_demo       # Social network analysis
cargo run --example knowledge_graph_demo      # Enterprise knowledge graph
cargo run --example enterprise_soc_demo       # Security operations center
cargo run --example agentic_enrichment_demo   # GAK (Generation-Augmented Knowledge)
cargo run --example persistence_demo          # Persistence & multi-tenancy
cargo run --example cluster_demo              # Raft clustering
cargo run --example ldbc_loader               # Load LDBC SNB SF1 dataset
cargo run --example finbench_loader           # Load/generate FinBench dataset
cargo run --release --example cricket_loader  # Load 21K Cricsheet matches
cargo run --release --example aact_loader     # Load AACT clinical trials dataset

# Start RESP server
cargo run                      # RESP on 127.0.0.1:6379, HTTP on :8080

# Code quality
cargo fmt -- --check           # Check formatting
cargo clippy -- -D warnings    # Lint checks

# Integration tests (requires running server)
cd tests/integration
python3 test_resp_basic.py
python3 test_resp_visual.py

# Frontend UI (React)
cd ui && npm install       # Install frontend dependencies
cd ui && npm run dev       # Dev server on :5173 (proxies /api to :8080)
cd ui && npm run build     # Production build → ui/dist/
```

## Architecture

### Module Structure

```
src/
├── graph/           # Property Graph Model
│   ├── store.rs     # GraphStore - in-memory storage with indices + cardinality stats
│   ├── node.rs      # Node with labels and properties
│   ├── edge.rs      # Directed edges with types
│   ├── property.rs  # PropertyValue (String, Integer, Float, Boolean, DateTime, Array, Map, Null)
│   └── types.rs     # NodeId, EdgeId, Label, EdgeType
│
├── query/           # OpenCypher Query Engine (~90% coverage)
│   ├── parser.rs    # Pest-based OpenCypher parser
│   ├── cypher.pest  # PEG grammar (atomic keyword rules for word boundaries)
│   ├── ast.rs       # Query AST
│   └── executor/
│       ├── planner.rs   # Query planner (AST → ExecutionPlan)
│       ├── operator.rs  # Physical operators (Volcano iterator model)
│       └── record.rs    # Record, RecordBatch, Value (with late materialization)
│
├── protocol/        # RESP Protocol
│   ├── resp.rs      # RESP3 encoder/decoder
│   ├── server.rs    # Tokio TCP server
│   └── command.rs   # GRAPH.* command handler
│
├── persistence/     # Persistence & Multi-Tenancy
│   ├── storage.rs   # RocksDB with column families
│   ├── wal.rs       # Write-Ahead Log
│   └── tenant.rs    # Multi-tenancy & resource quotas
│
├── raft/            # High Availability
│   ├── node.rs      # RaftNode using openraft
│   ├── state_machine.rs  # GraphStateMachine
│   ├── cluster.rs   # ClusterConfig, ClusterManager
│   ├── network.rs   # Inter-node communication
│   └── storage.rs   # Raft log storage
│
├── nlq/             # Natural Language Query Pipeline
│   ├── mod.rs       # NLQPipeline (text_to_cypher, extract_cypher, is_safe_query)
│   └── client.rs    # NLQClient (OpenAI, Gemini, Ollama, Claude Code providers)
│
├── vector/          # HNSW Vector Index
├── snapshot/        # Portable .sgsnap export/import
└── sharding/        # Tenant-level sharding
```

### Key Architectural Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fab679/graphmind](https://github.com/fab679/graphmind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
