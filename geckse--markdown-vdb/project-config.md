---
trigger: always_on
description: A filesystem-native vector database built around Markdown files. Rust, zero infrastructure, optimized for AI agents.
---

# Markdown VDB

A filesystem-native vector database built around Markdown files. Rust, zero infrastructure, optimized for AI agents.

All 18 implementation phases plus graph-enhanced search are complete and passing (612 tests, clippy clean).

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Agent Interface                       │
│              CLI (clap) + Library API                    │
│         mdvdb search | ingest | status | watch          │
├──────────┬──────────┬───────────┬───────────────────────┤
│  Search  │  Schema  │ Clustering│   File Watcher        │
│  Engine  │  System  │  (linfa)  │   (notify)            │
├──────────┴──────────┴───────────┴───────────────────────┤
│                   Index Storage                         │
│        usearch (HNSW) + rkyv (metadata) + memmap2       │
│          parking_lot::RwLock (concurrency)               │
├──────────┬──────────────────────────────────────────────┤
│ Embedding│   OpenAI | Ollama | Custom (reqwest)         │
│ Providers│   Batch processing + content-hash skip       │
├──────────┴──────────────────────────────────────────────┤
│                  Chunking Engine                        │
│      Heading-split + token size guard (tiktoken-rs)     │
├─────────────────────────────────────────────────────────┤
│              Markdown Parsing & Discovery                │
│    pulldown-cmark + serde_yaml + ignore + sha2          │
├─────────────────────────────────────────────────────────┤
│               Foundation & Configuration                │
│         dotenvy + thiserror + anyhow + tracing          │
└─────────────────────────────────────────────────────────┘
```

## Project Structure

```
src/
├── main.rs              # CLI entry point (clap + anyhow)
├── lib.rs               # Public library API (MarkdownVdb)
├── config.rs            # Config loading: shell env → project → .env → user → defaults
├── format.rs            # Human-readable output formatting (colors, bars, timestamps)
├── error.rs             # Error enum (thiserror)
├── logging.rs           # Tracing subscriber setup
├── discovery.rs         # File scanning with ignore patterns (.gitignore + .mdvdbignore)
├── parser.rs            # Markdown parsing: frontmatter, headings, body
├── chunker.rs           # Heading-based chunking + token size guard
├── search.rs            # Query pipeline, metadata filtering, time decay, graph expansion, results
├── fts.rs               # Full-text search (Tantivy BM25 wrapper)
├── links.rs             # Link graph extraction, backlinks, orphan detection, multi-hop BFS, neighborhood
├── tree.rs              # File tree with sync status indicators
├── schema.rs            # Auto-infer + overlay schema system
├── clustering.rs        # K-means, nearest-centroid, rebalancing, TF-IDF labels
├── watcher.rs           # Filesystem watcher (notify + debouncer)
├── ingest.rs            # Full + incremental ingestion pipeline
├── embedding/
│   ├── mod.rs           # EmbeddingProvider trait + factory
│   ├── provider.rs      # Trait definition
│   ├── openai.rs        # OpenAI-compatible provider
│   ├── ollama.rs        # Ollama provider
│   ├── batch.rs         # Concurrent batch orchestration (up to 4) + hash skip
│   └── mock.rs          # Mock provider for testing
└── index/
    ├── mod.rs           # Index public API
    ├── types.rs         # StoredChunk, StoredFile, IndexMetadata (rkyv)
    ├── storage.rs       # File I/O: header + rkyv region + usearch region
    └── state.rs         # Runtime operations with RwLock concurrency

tests/
├── api_test.rs          # Library API integration tests
├── cli_test.rs          # CLI binary integration tests
├── chunker_test.rs      # Chunking pipeline tests
├── clustering_test.rs   # K-means clustering tests
├── config_test.rs       # Configuration loading tests
├── discovery_test.rs    # File discovery tests
├── embedding_test.rs    # Embedding provider tests
├── fts_test.rs          # Full-text search (Tantivy BM25) tests
├── graph_test.rs        # Graph traversal + multi-hop search tests
├── index_test.rs        # Index storage + mtime tests
├── ingest_test.rs       # Ingestion pipeline tests
├── links_test.rs        # Link graph + backlinks tests
├── parser_test.rs       # Markdown parsing tests
├── schema_test.rs       # Schema inference tests
├── search_test.rs       # Search engine + time decay tests
├── tree_test.rs         # File tree tests
└── watcher_test.rs      # File watcher tests

docs/prds/               # PRD specifications for all 18 phases (reference)
```

## Core Design Decisions

- **Config:** Dotenv-style files, NOT TOML/YAML. Resolution: shell env > `.markdownvdb/.config` > `.markdownvdb` (legacy) > `.env` > `~/.mdvdb/config` (user) > defaults
- **Index directory:** `.markdownvdb/` contains `index` (binary: `[64B header][rkyv metadata][usearch HNSW]`) + `fts/` (Tantivy BM25 segments). Configured via `MDVDB_INDEX_DIR`.
- **Paths:** ALL file paths in the index are relative to project root. Never absolute.
- **Errors:** `thiserror` for typed library errors, `anyhow` only at CLI boundary in `main.rs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geckse/markdown-vdb](https://github.com/geckse/markdown-vdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
