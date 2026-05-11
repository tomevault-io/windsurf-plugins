---
trigger: always_on
description: Cashew is a thought-graph memory engine for AI agents. It stores knowledge as nodes in a SQLite graph, connects them with edges, and retrieves relevant context via recursive BFS over organic graph structure. Everything lives in one SQLite file — no external servers, no separate indexes.
---

# CLAUDE.md — Cashew Developer Guide

Cashew is a thought-graph memory engine for AI agents. It stores knowledge as nodes in a SQLite graph, connects them with edges, and retrieves relevant context via recursive BFS over organic graph structure. Everything lives in one SQLite file — no external servers, no separate indexes.

**IMPORTANT**: Cashew does NOT call LLMs directly. It is purely the BRAIN (storage + retrieval + structure). The PROCESSOR (LLM) is external and provided by the orchestrator (OpenClaw) via `model_fn` parameters.

## Architecture

```
scripts/cashew_context.py  — Main CLI entry point (context, extract, think, sleep, stats)
cashew_cli.py              — Secondary CLI (init, install-crons, ingest)
core/                      — Engine modules
core/extractors.py         — Extractor plugin interface (BaseExtractor, ExtractorRegistry)
extractors/                — Built-in source extractors (obsidian, sessions, markdown)
extractors/utils.py        — Shared extractor utilities (frontmatter, wikilinks, ignore patterns)
integration/               — OpenClaw bridges
```

### Core Modules

| Module | Purpose |
|--------|---------|
| `config.py` | YAML config loading with env var expansion. Global `config` singleton. |
| `embeddings.py` | Sentence-transformer embeddings (all-MiniLM-L6-v2). sqlite-vec virtual table for O(log N) search. `search()`, `check_novelty()`, `embed_text()`. |
| `retrieval.py` | `retrieve_recursive_bfs()` — seeds via sqlite-vec, BFS graph walk, per-hop scoring. The sole retrieval method. |
| `context.py` | Composes retrieval results into formatted context strings for LLM consumption. |
| `session.py` | Session lifecycle: `start_session()`, `end_session()`, `think_cycle()`, `tension_detection()`. |
| `sleep.py` | Deep consolidation: cross-linking, decay, deduplication, core memory promotion. Runs daily. |
| `decay.py` | Prunes stale nodes (low access, low confidence, old). Sets `decayed=1`. |
| `traversal.py` | Graph walk utilities (DFS, BFS, path finding). |
| `stats.py` | Graph statistics and health metrics. |
| `export.py` | Export graph data to JSON for dashboard visualization. |
| `extractors.py` | Plugin interface: `BaseExtractor` (ABC), `ExtractorRegistry` (register, run, state persistence, dedup). |
| `graph_utils.py` | Shared utilities: `load_embeddings()`, `cosine_similarity()`. |

### Extractors

Built-in source extractors in `extractors/`:

| Module | Purpose |
|--------|---------|
| `obsidian.py` | Obsidian vault extraction. Frontmatter, `[[wikilink]]` edges, `.obsidianignore`, domain from folder structure. |
| `sessions.py` | OpenClaw session JSONL extraction. Incremental line tracking, filters tool calls/system messages. |
| `markdown_dir.py` | General markdown directory extraction. `.cashewignore` support. |
| `utils.py` | Shared: `parse_frontmatter()`, `extract_wikilinks()`, `load_ignore_patterns()`, `should_ignore()`, `split_into_paragraphs()`, `detect_domain_from_path()`. |

All extractors: use `model_fn` for LLM extraction when available, fall back to paragraph splitting when not. Checkpointing via `get_state()`/`set_state()` persisted automatically by the registry.

### Integration

| Module | Purpose |
|--------|---------|
| `integration/openclaw.py` | OpenClaw agent lifecycle hooks. `generate_session_context()`, `extract_from_conversation()`, `run_think_cycle()`. |

## Database Schema

SQLite with 3 tables + 1 virtual table:

- **`thought_nodes`** — Knowledge nodes (id, content, node_type, domain, confidence, access_count, decayed, permanent, tags)
- **`derivation_edges`** — Relationships (parent_id, child_id, weight, confidence)
- **`embeddings`** — Vector embeddings per node (node_id, vector as BLOB, model name)
- **`vec_embeddings`** — sqlite-vec virtual table for O(log N) nearest neighbor search (node_id, embedding float[384], cosine distance)

### Key Columns

- `domain` — Classifies who the knowledge belongs to. Configurable via config.yaml.
- `node_type` — One of: fact, observation, insight, decision, belief, derived, meta, core_memory, cross_link
- `decayed` — 0 or 1. Decayed nodes are excluded from retrieval but kept in DB.
- `permanent` — 0 or 1. Permanent nodes are immune to decay.
- `confidence` — Float 0-1. Think cycle outputs are gated at 0.75.
- `tags` — Comma-separated tags (e.g., `vault:private` for privacy classification).

## Retrieval: Recursive BFS

The sole retrieval method. No hotspots, no hierarchy, no DFS.

1. **Seed selection** — Embed query, find top-k nearest nodes via `vec_embeddings MATCH` (O(log N) with sqlite-vec, brute-force fallback)
2. **BFS traversal** — From seeds, explore graph neighbors up to `max_depth` hops. At each hop, score neighbors by cosine similarity, keep top `picks_per_hop`.
3. **Final ranking** — All candidates scored and sorted by similarity.

Parameters: `n_seeds=5`, `picks_per_hop=3`, `max_depth=3`.

The graph's organic connectivity (built by sleep cycle cross-linking) provides implicit hierarchy. No synthetic summary nodes needed.

## sqlite-vec Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rajkripal/cashew](https://github.com/rajkripal/cashew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
