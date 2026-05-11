---
trigger: always_on
description: Tiered semantic memory system for AI agents using HNSW-based vector indexing.
---

# Sem-Mem Project

Tiered semantic memory system for AI agents using HNSW-based vector indexing.

## Recent Improvements

### Performance & Efficiency

These optimizations reduce API costs, latency, and I/O overhead:

- **Batch embeddings** - `bulk_learn_pdf()` and `save_thread_to_memory()` use a single `embed()` batch call instead of one embedding API call per chunk. A 50-chunk PDF makes 1 API call instead of 50.

- **Query expansion caching** - `_expand_query()` results are cached in an LRU cache (128 entries). Repeated queries skip the LLM call entirely.

- **Deferred index saves** - `HNSWIndex` tracks a `_dirty` flag. The new `flush()` method only writes to disk when changes exist, preventing redundant full-file rewrites during rapid ingestion (PDF upload, batch import, consolidation).

- **Vectorized L1 cache search** - L1 similarity uses `query_matrix @ cache_matrix.T` (BLAS-accelerated matrix multiply) instead of per-item `np.dot()` loops. O(1) NumPy call replaces O(cache_size x num_queries) Python loop.

- **Batch consolidation dedup** - `_batch_check_patterns_exist()` embeds all candidate patterns in one batch call and searches HNSW once per pattern, replacing N separate `recall()` calls (each with its own embedding + optional query expansion).

- **Query expansion opt-in** - `recall(expand_query=...)` defaults to `False`. Only user-facing `chat_with_memory()` passes `expand_query=True`. Internal callers (consolidation, dedup, auto-memory) no longer pay the LLM overhead.

- **Async client reuse** - `AsyncSemanticMemory.save_memory()` lazy-initializes a reusable sync OpenAI client instead of creating a new one per call.

### Inspectability & Trust

These changes make Sem-Mem more inspectable, trustworthy, and production-friendly: you can see how memory evolved, why certain facts were preferred, and what each background pass actually did.

- **Structured progress logging** - Each consolidation run records what changed and why in a machine-readable log (`progress_log.jsonl`), making long-running memory evolution auditable and debuggable.

- **Project manifests & progress summaries** - Per-project / per-thread "manifest" and "what we did / what's next" memories (`save_project_manifest()`, `append_thread_progress()`) make it easy for both humans and agents to re-enter complex work without re-reading long threads.

- **Smarter status & progress queries** - Queries like "what's the status of X?" preferentially surface manifests and progress logs, so you get a concise state-of-the-world answer instead of raw history dumps.

- **Explicit conflict handling & precedence** - Conflicting memories (fact vs. correction, old vs. new policy) are handled via clear rules: explicit correction entries, recency and utility bias, and no silent overwrites. See [Conflict Handling & Precedence](#conflict-handling--precedence).

- **Contradiction surfacing, not silent fixes** - Detected contradictions are logged (with memory IDs and summaries) to `contradictions.json` for human review, providing a clear trail of "what disagrees with what" instead of hidden heuristics.

- **Stronger guarantees about consolidation behavior** - Consolidation is explicitly offline, human-scheduled, and non-agentic, with hard caps on changes per run (`CONSOLIDATION_MAX_NEW_PATTERNS`) and clear configuration describing its role.

## Architecture

- **SmartCache (L1)**: Segmented LRU in RAM with Protected/Probation tiers
- **L2 Storage**: HNSW index (`hnsw_index.bin` + `hnsw_metadata.json`) for O(log n) semantic search
- **Embeddings**: Pluggable embedding providers (OpenAI, local via sentence-transformers, Ollama)
- **Instructions**: Persistent system instructions in `local_memory/instructions.txt`
- **UI**: Streamlit app (`app.py`)

## Embedding Providers

Sem-Mem supports multiple embedding providers. The recommended setup is local embeddings via sentence-transformers (no API costs, no rate limits).

### Available Providers

| Provider | Model | Dimension | API Key Required |
|----------|-------|-----------|------------------|
| `sentence-transformers` (recommended) | Qwen/Qwen3-Embedding-0.6B | 1024 | No |
| `local` (alias) | Qwen/Qwen3-Embedding-0.6B | 1024 | No |
| `openai` | text-embedding-3-small | 1536 | Yes |
| `ollama` | nomic-embed-text | 768 | No |
| `google` | text-embedding-004 | 768 | Yes |

### Switching to Local Embeddings

To switch from OpenAI to local embeddings (recommended to avoid rate limits):

```bash
# 1. Install sentence-transformers
pip install sentence-transformers torch

# 2. Run migration script (backs up existing index, re-embeds all memories)
python scripts/migrate_embeddings.py --provider sentence-transformers

# 3. Update your .env file
SEMMEM_EMBEDDING_PROVIDER=sentence-transformers

# 4. Restart your application
```

### Configuration

Set embedding provider via environment variables:

```bash
# In .env file or environment
SEMMEM_EMBEDDING_PROVIDER=sentence-transformers  # or: openai, local, ollama, google
SEMMEM_EMBEDDING_MODEL=Qwen/Qwen3-Embedding-0.6B  # optional, uses provider default

# Chat still uses OpenAI (or other provider)
SEMMEM_CHAT_PROVIDER=openai
```

Or programmatically:

```python
from sem_mem import SemanticMemory

# Use local embeddings with OpenAI chat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrDavidL/sem-mem](https://github.com/DrDavidL/sem-mem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
