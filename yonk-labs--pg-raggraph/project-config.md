---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**pg-raggraph** is a PostgreSQL-native GraphRAG library. It implements knowledge graph construction, entity extraction, and hybrid retrieval (vector similarity + graph traversal) using PostgreSQL as the single backing store — no separate graph database or vector database required.

**Core thesis:** pgvector (vector search) + adjacency tables + recursive CTEs (graph traversal) + PostgreSQL full-text search (BM25) = a complete GraphRAG stack in one ACID-compliant database. No graph database extensions required.

## Architecture

### PostgreSQL Extensions Required
- **pgvector** — vector similarity search (HNSW/IVFFlat indexes)
- **pg_trgm** — trigram fuzzy matching for entity resolution

### Why NOT Apache AGE
We evaluated AGE extensively (see `research/apache-age-evaluation.md`) and decided against it:
- **Cloud compatibility** — AGE requires `shared_preload_libraries` (PG restart). Only Azure supports it among major managed providers. No AWS RDS, no GCP Cloud SQL, no Supabase, no Neon. This kills adoption.
- **Awkward pgvector composition + Cypher subset** — AGE `cypher()` calls CAN be composed with pgvector in one SQL statement (Microsoft's HorizonDB GraphRAG doc shows the CTE pattern, verified on stock AGE 1.5.0: https://learn.microsoft.com/en-us/azure/horizondb/ai/graph-rag). But the pattern is undocumented outside Azure's fork, needs agtype casting at every boundary, and per Microsoft's own doc AGE implements only a subset of openCypher (no `MERGE ... ON CREATE SET`, `EXISTS`, `datetime()`), has manual graph maintenance with no CDC, and exhibits exponential path expansion beyond 3-4 hops. Recursive CTEs are plain SQL with none of that.
- **Performance** — In the preregistered cap-gold-v1 head-to-head (`benchmarks/age-bakeoff/cap-gold-v1/PIPELINES.md`, bare SQL both engines, exact anchors, row parity verified): recursive CTEs held sub-ms p50 through 3 hops vs AGE variable-length Cypher 33.7→132.1 ms p50 (670 ms p95 at 3 hops); ~8x vs AGE's cheaper fixed-hop form at 1 hop. Earlier measurements (2-40x pre-build research, 42-101x bake-off adapter-level) are consistent but harness-scoped. Honest counterweight: AGE's one-row-per-doc shape can win 1-hop semantic-first pipeline latency (9.5 vs 37 ms) — our chunk granularity costs those ms and buys the recall win (R@20 0.133 vs 0.084). AGE has also produced catastrophic query plans (LightRAG issue #2255: 49 billion estimated rows, 17-hour migration).
- **Proven alternative** — postgres-graph-rag already validates that adjacency tables + recursive CTEs + pgvector is sufficient for full GraphRAG.

Our graph approach: **adjacency tables** (`entities` + `relationships`) with proper indexes, **recursive CTEs** for multi-hop traversal, all composable with pgvector in unified SQL queries.

### Design Principles
- **PostgreSQL-first** — not a storage adapter bolted onto another framework. SQL is the first-class query interface.
- **Single database** — all data (documents, chunks, entities, relationships, embeddings, community summaries, provenance metadata) lives in one PostgreSQL instance.
- **Async Python** — native async with connection pooling (asyncpg or psycopg3 async).
- **Incremental updates** — graph updates on document change without full re-indexing. Track ingested documents via content hashes.
- **Pluggable LLM providers** — entity extraction and embedding generation work with any provider (OpenAI, Anthropic, Ollama, etc.).

### Key Subsystems
1. **Ingestion pipeline** — document chunking → embedding generation → entity/relationship extraction (LLM, parallel via asyncio semaphores) → entity resolution (pg_trgm fuzzy + vector dedup) → graph storage in a single per-document transaction
2. **Graph store** — adjacency tables (`entities` + `relationships`) with JSONB properties, recursive CTEs for multi-hop traversal, proper indexes on src_id/dst_id for fast joins
3. **Hybrid retrieval** — combines pgvector cosine similarity with recursive CTE graph traversal and BM25 (`to_tsquery` with OR semantics) full-text search in unified SQL queries
4. **Smart mode** (default) — confidence-triggered routing. Runs naive first; ships if confidence ≥ 0.7, applies cheap 1-hop graph boost if 0.4-0.7, escalates to full hybrid if < 0.4
5. **Chunking** — auto-detects markdown (heading-aware), code (function/class boundaries for Python/JS/TS/Go/Rust), or plain text (sentence-aware). Hard token-split fallback prevents oversized chunks from overflowing LLM context
6. **Ingestion profiles** — `conservative` / `balanced` (default) / `aggressive` / `max` control `doc_concurrency` and `extract_concurrency` for CPU budget
7. **`pgrg devmem`** — convenience CLI for developer knowledge bases with dev-tuned extraction prompt (person/service/library/file/commit/incident/ADR entities) and code-aware chunking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yonk-labs/pg-raggraph](https://github.com/yonk-labs/pg-raggraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
