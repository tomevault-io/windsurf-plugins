---
trigger: always_on
description: Drop this file into your project root or reference it from your Claude Code config to teach Claude how to use its persistent memory layer.
---

# Dragon Brain — CLAUDE.md

Drop this file into your project root or reference it from your Claude Code config to teach Claude how to use its persistent memory layer.

## The Harness

**Assertive pushback is non-negotiable. See global `~/.claude/CLAUDE.md` § The Harness — 6 guardrails against yes-man behavior. Do NOT hedge-then-agree. Say NO first, make Tabish argue his case. Devil's advocate on every decision. Name the opportunity cost.**

## What This Is

A persistent memory system for AI agents. Knowledge graph (FalkorDB) + vector search (Qdrant) + MCP server. Any MCP-compatible client can store entities, observations, and relationships — then recall them semantically across sessions. Published on PyPI as `dragon-brain`. **v1.1.0 — 100% recall@5 on LongMemEval (ICLR 2025), no LLM required.**

## Current Architecture

6-channel parallel retrieval pipeline, all channels fire on every query:
- **Dense vector** (Qdrant, BGE-M3 1024d) — semantic similarity
- **FTS5 lexical** (SQLite BM25) — keyword matches embeddings miss
- **Entity-first** (spaCy NER → FalkorDB graph) — entity → MENTIONED_IN → sessions
- **Temporal** (date parser → timeline query) — time-window filtering
- **Relational** (graph traversal) — shared entity connections
- **Associative** (spreading activation) — energy propagation through graph

Fusion: weighted RRF (k=35, PIT percentile normalization). Intent classifier sets per-channel weights (soft routing, no hard gate). Optional cross-encoder reranking (ms-marco-MiniLM, GPU/CPU auto-detect).

## Setup Verification

At the start of every session, verify the memory system is running:

```
docker ps --filter "name=claude-memory"
```

You should see 4 healthy containers: graphdb, qdrant, embeddings, dashboard.

If MCP tools (`search_memory`, `create_entity`, etc.) are not available, the server may need restarting. MCP failures are **silent** — always verify tool availability at session start.

## Updating

```bash
cd claude-memory-mcp
git pull origin master
pip install -e ".[dev]"
```

If Docker images changed: `docker compose pull && docker compose up -d`

## How to Search

### Default (Hybrid Search — Recommended)

```
search_memory(query="your question here")
```

No strategy parameter needed. The default path:
1. Runs vector similarity search (always)
2. Detects query intent (temporal, relational, associative, or semantic)
3. Enriches results with graph signals based on detected intent
4. Merges via Reciprocal Rank Fusion if graph results found entities that vector search missed

### Explicit Strategies (When You Know What You Want)

| Strategy | When to Use | Example |
|----------|-------------|---------|
| `"semantic"` | Pure meaning-based similarity | `search_memory(query="distributed systems", strategy="semantic")` |
| `"temporal"` | Time-based queries | `search_memory(query="last week's work", strategy="temporal")` |
| `"relational"` | Path/connection queries (quote entity names) | `search_memory(query="path between \"Auth\" and \"Database\"", strategy="relational")` |
| `"associative"` | Spreading activation through the graph | `search_memory(query="related to authentication", strategy="associative")` |

### Temporal Window

Temporal queries default to a 7-day lookback. Widen if needed:

```
search_memory(query="recent progress", temporal_window_days=14)
```

Use `include_meta=True` to see if there are more results beyond the window:

```
search_memory(query="recent work", include_meta=True)
```

If the response includes `"temporal_exhausted": true`, widen the window for more history.

### Understanding Results

Each result includes:

| Field | Meaning |
|-------|---------|
| `score` | Primary ranking score (cosine similarity, RRF composite, or activation composite) |
| `retrieval_strategy` | What generated this result: `"semantic"`, `"hybrid"`, `"temporal"`, `"relational"`, `"associative"` |
| `vector_score` | Raw cosine similarity from Qdrant. `null` if entity had no vector match |
| `recency_score` | 0-1 exponential decay. 1.0 = just created, 0.5 = ~7 days old |
| `activation_score` | Spreading activation energy (associative results only) |
| `path_distance` | Graph hops from query anchor (relational results only) |
| `salience_score` | Entity importance/frequency score |

**Key insight:** If `score` is 0.0, check `retrieval_strategy` — it tells you why. A temporal-only result with no vector embedding will legitimately have `score=0.0` and `vector_score=null`.

## How to Store Memories

### Entities (Things)

```
create_entity(name="Project Alpha", node_type="Entity", project_id="my-project")
```

Common node types: `Entity`, `Concept`, `Session`, `Breakthrough`, `Tool`, `Decision`, `Bottle`, `Analogy`, `Issue`, `Project`, `Procedure`, `Person`

### Observations (Facts About Things)

```
add_observation(entity_id="<uuid>", content="This project uses a microservices architecture")
```

Observations are automatically embedded and indexed for semantic search. **Adding an observation also re-embeds the parent entity** — entity vectors include observation content for richer semantic matching (not just name/type/description).

### Relationships (Connections)

```
create_relationship(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iikarus/Dragon-Brain](https://github.com/iikarus/Dragon-Brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
