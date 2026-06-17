---
trigger: always_on
description: > Machine-readable project context for AI coding assistants (Claude Code, Cursor, Copilot, etc.)
---

# AGENTS.md — VelociRAG for AI Coding Agents

> Machine-readable project context for AI coding assistants (Claude Code, Cursor, Copilot, etc.)

## What Is This?

VelociRAG is **lightning-fast RAG for AI agents**. Pure retrieval engine powered by ONNX Runtime with 4-layer fusion, 3ms warm embeddings, MCP server, and Unix socket daemon.

- **Language:** Python 3.10+
- **Backend:** ONNX Runtime (no PyTorch)
- **Source:** `src/velocirag/` (18 modules, ~12K lines)
- **Tests:** `tests/` (18 test files)
- **CLI:** `velocirag` (click-based)
- **License:** MIT

## Architecture

```
markdown files → chunk → embed (ONNX) → store (SQLite + FAISS)
                                            ↓
                                      4-layer search:
                                        vector (FAISS cosine, 384d MiniLM-L6-v2)
                                      + keyword (BM25 via SQLite FTS5)
                                      + graph (knowledge graph traversal)
                                      + metadata (structured SQL filters)
                                            ↓
                                      RRF fusion → cross-encoder rerank → results
```

**Three ways to query:**
1. **MCP server** (`velocirag mcp`) — for AI agents via Model Context Protocol
2. **Daemon** (`velocirag serve`) — warm engine over Unix socket for CLI users
3. **Direct** (`velocirag search`) — cold search, no daemon needed

## Module Map

| Module | Lines | Purpose |
|--------|-------|---------|
| `cli.py` | 1595 | Click CLI — index, search, serve, stop, status, mcp, health, query, reindex. Cascading delete orchestration on file cleanup. |
| `analyzers.py` | 1568 | 7 graph analyzers + FAISS semantic (128-token truncation, skip <50 char docs) + sampled centrality. |
| `pipeline.py` | 1275 | 10-stage graph build. Incremental updates with file-centric provenance. `final_nodes`/`final_edges` in all return paths. |
| `store.py` | 1197 | Vector storage — SQLite + FAISS + FTS5. Batched rebuild. Cascading deletes with conn passthrough. Empty FAISS persistence. |
| `graph.py` | 1116 | Knowledge graph — Node/Edge models, GraphStore (SQLite), GraphQuerier. `remove_by_source_file()` with orphan pruning. |
| `unified.py` | 901 | 4-layer fusion search — vector + keyword + metadata + graph → RRF. Filename cache. Exact-match promotion. |
| `metadata.py` | 744 | Metadata store — frontmatter, tags, cross-refs, usage tracking. `remove_document()` with orphan tag pruning. |
| `searcher.py` | 688 | High-level search — query variants, batch FAISS, RRF fusion, consistency validation |
| `embedder.py` | 541 | ONNX Runtime embeddings (all-MiniLM-L6-v2, 384d). 3ms warm, 184ms cold. |
| `mcp_server.py` | 499 | FastMCP server — 5 tools. Thread-safe init, `threading.Event`. |
| `daemon.py` | 462 | Unix socket search daemon — warm engine, bounded queue, auto-detected by CLI |
| `tracker.py` | 305 | Usage tracking — search hits, reads, access patterns |
| `reranker.py` | 235 | Cross-encoder reranking (TinyBERT via ONNX). Lazy init. |
| `variants.py` | 217 | Query variant generation + acronym registry + question rewrite |
| `chunker.py` | 177 | Markdown chunking by headers with parent context preservation |
| `frontmatter.py` | 172 | YAML frontmatter parser, tag extraction, wiki-link extraction |
| `rrf.py` | 144 | Reciprocal Rank Fusion — shallow copy in hot path |

## Key Classes

```python
# Index documents
embedder = Embedder()  # ONNX Runtime, lazy model download
store = VectorStore("./db", embedder)
store.add_directory("./my-notes", source="notes")

# Basic search
searcher = Searcher(store, embedder)
results = searcher.search("query", limit=5, threshold=0.3)

# Full unified search (4 layers)
graph_store = GraphStore("./db/graph.db")
metadata_store = MetadataStore("./db/metadata.db")
unified = UnifiedSearch(searcher, graph_store, metadata_store)
results = unified.search("query", limit=5, enrich_graph=True)

# Build knowledge graph
pipeline = GraphPipeline(graph_store, embedder, metadata_store, entity_extractor="gliner")
pipeline.build("./my-notes", force_rebuild=True)

# BM25 keyword search (direct)
results = store.keyword_search("exact phrase", limit=10)

# Graph queries
querier = GraphQuerier(graph_store)
querier.find_connections("node_title", depth=2)
querier.get_topic_web("topic")
querier.get_hub_nodes(limit=10)

# Cascading deletes (v0.6.3+)
# When files are deleted, cleanup cascades across all stores:
store._cleanup_deleted_files(Path('./docs'), 'notes')  # vector + FTS5
metadata_store.remove_document('deleted-file.md')       # metadata + orphan tags
graph_store.remove_by_source_file('/abs/path/file.md')  # nodes + edges + orphans

# Daemon client
from velocirag.daemon import daemon_search, daemon_ping
if daemon_ping():
    results = daemon_search("query", limit=5)
```

## CLI Commands

```bash
velocirag index <path> [--db PATH] [--graph] [--metadata] [--gliner] [--light-graph] [--force]
velocirag search <query> [--db PATH] [--limit N] [--threshold F] [--format text|json]
velocirag serve [--db PATH] [-f]       # start search daemon
velocirag stop                          # stop daemon
velocirag status                        # daemon health
velocirag mcp [--db PATH] [--transport stdio|sse]  # MCP server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaseebKhalid1507/VelociRAG](https://github.com/HaseebKhalid1507/VelociRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
