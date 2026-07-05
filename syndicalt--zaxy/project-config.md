---
trigger: always_on
description: Markdown files + vector DBs are the dominant approach for agent persistent context, but they are fundamentally lossy and inefficient:
---

# Zaxy: Event-Sourced Temporal Knowledge Graph Fabric

## Problem Statement

Markdown files + vector DBs are the dominant approach for agent persistent context, but they are fundamentally lossy and inefficient:

- **No relational reasoning**: Vector similarity can't do multi-hop traversal or follow causal chains.
- **No temporal awareness**: Can't answer "What was true then vs. now?" — facts overwrite each other silently.
- **Non-replayable**: Context is chunked and flattened; you can't reconstruct how the agent arrived at a decision.
- **Un-auditable**: No provenance chain for compliance or debugging.

> **Current architecture (read this first).** The controlling architecture docs
> are [docs/architecture.md](docs/architecture.md) and [README.md](README.md).
> The default projection backend is the **embedded LadybugDB** store (in-process,
> no external service); Neo4j, pgGraph, and LatticeDB are optional backends
> selected via `PROJECTION_BACKEND`, behind one pluggable projection contract.
> Some ADRs below predate the embedded-default move and are kept as historical
> decision records — where an older note says "Neo4j", read "the projection
> backend (embedded by default)".

## Architecture Decision Record

### ADR-1: Event-Sourced Foundation

**Decision**: Use Eventloom's append-only JSONL as the immutable source of truth.

**Rationale**:
- Hash-chain integrity (tamper-evident).
- Deterministic replay.
- Zero write overhead (local file append).
- Cross-process locking already solved.

**Trade-off**: Single-writer per file. For multi-agent distributed setups, shard by session or add a log aggregation layer later.

### ADR-2: Hybrid Extraction (Rule-Based + LLM Fallback)

**Decision**: Extract entities/relations from events using registered rule-based extractors first, LLM fallback only for unstructured events.

**Rationale**:
- Eventloom events are strongly typed (`goal.created`, `task.proposed`, etc.).
- Typed events map deterministically to graph schema.
- Reduces LLM extraction cost by 60–80%.
- Faster ingestion (<50ms vs 500ms–2s for LLM).

**Trade-off**: New event types require writing an extractor. This is intentional — it forces schema discipline.

### ADR-3: Direct Neo4j Cypher vs. Graphiti Abstraction

**Decision**: Use the official `neo4j` Python driver with custom Cypher rather than Graphiti's high-level `add_episode` API.

**Rationale**:
- Full control over bi-temporal schema (`valid_from`, `valid_to`).
- Our extraction engine already produces structured `ExtractedEntity`/`ExtractedEdge` objects.
- Graphiti's LLM-based extraction is redundant with our hybrid extractor.
- Graphiti's hybrid search (vector + BM25 + traversal) can be replicated with native Neo4j indexes.

**Trade-off**: We maintain more Cypher. Mitigated by keeping queries simple and tested.

**Update**: the projection layer is now a pluggable contract. Embedded LadybugDB
is the default backend; Neo4j-direct-Cypher is one optional backend among several
(pgGraph, LatticeDB).

### ADR-4: Hybrid Retrieval (Exact + Keyword + Traversal)

**Decision**: Query router fuses three strategies with configurable weights.

**Rationale**:
- **Exact**: Fast lookup when the query is an entity name.
- **Keyword/BM25**: Full-text for semantic similarity on names/summaries.
- **Traversal**: Multi-hop expansion from top keyword hits.
- Each covers blind spots of the others.

**Trade-off**: Fusion adds ~10–50ms latency. Acceptable for agent context quality.

### ADR-5: Pathlight for Observability (Not Storage)

**Decision**: Pathlight can trace every memory operation when enabled, but does not store context itself.

**Rationale**:
- Eventloom = durable history.
- Projection backend (embedded LadybugDB by default) = structured reasoning layer.
- Pathlight = execution tracing + breakpoints + diff.
- Clean separation of concerns.

**Trade-off**: Extra network call per traced operation. Mitigated by async batching and optional disabling.

### ADR-6: MCP as Primary Interface

**Decision**: Expose memory via MCP tools (`memory_append`, `memory_query`, `memory_replay`, `memory_invalidate`).

**Rationale**:
- Framework-agnostic (LangGraph, CrewAI, AutoGen, Claude Desktop, etc.).
- Standardized schema discovery and type safety.
- One-click integration via `mcpServers` config.

**Trade-off**: Requires MCP client support. Major frameworks already have it (2025+).

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Python | 3.11+ |
| Graph projection (default) | embedded LadybugDB | pinned |
| Optional graph backends | Neo4j / pgGraph / LatticeDB | — |
| Validation | Pydantic | 2.7+ |
| MCP Server | mcp (official Python SDK) | 1.0+ |
| Observability | pathlight (Python SDK) | 0.1+ |
| CLI | typer | 0.12+ |
| Testing | pytest + pytest-asyncio + pytest-cov | 8.0+ |
| Lint/Format | ruff | 0.4+ |
| Types | mypy | 1.10+ |

## Directory Structure

```
zaxy/
├── pyproject.toml              # Project config, deps, tool settings
├── docker-compose.yml          # Neo4j + test services
├── AGENTS.md                   # This file
├── src/zaxy/
│   ├── __init__.py             # Public API exports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syndicalt/zaxy](https://github.com/syndicalt/zaxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
