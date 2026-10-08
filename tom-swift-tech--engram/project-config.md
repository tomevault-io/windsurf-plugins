---
trigger: always_on
description: > *A memory trace physically encoded in neural tissue that strengthens with reinforcement.*
---

# Engram — Memory Traces for AI Agents

> *A memory trace physically encoded in neural tissue that strengthens with reinforcement.*

## What This Is

Engram is a lightweight, zero-infrastructure memory system for AI agents that **learns** over time. Each agent gets its own SQLite file — an engram — containing everything it knows, has experienced, believes, and has learned. Raw facts are retained, entities and relationships are extracted, and periodic reflection cycles synthesize higher-order observations and confidence-scored beliefs.

Agents running on Engram don't just remember. They form opinions, refine them with evidence, and get smarter the longer they run.

## Repo

`swift-innovate/engram` — Apache 2.0

## Design Heritage

Engram synthesizes ideas from four prior efforts:

- **Hindsight** (vectorize-io): Four memory networks (world/experience/observation/opinion), retain/recall/reflect operations, bank-level configuration, confidence-scored beliefs. We took the cognitive architecture and dropped the Postgres dependency.
- **memory-ragkg** (our OpenClaw plugin design): SQLite-based knowledge graph with entities/relations tables, trust/provenance layer, hybrid query router. We carried the trust layer forward — neither Hindsight nor Memvid has it.
- **Hearthmind/Mira-Memory**: Lessons learned from mem0 integration pain — Ollama connectivity issues, write latency blocking agents, metadata restrictions. The fast-write/slow-extract pattern exists because of this.
- **Memvid**: Single-file portability inspiration. One file per agent, git-committable, zero infrastructure.

## Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                           ENGRAM                                  │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  RETAIN (fast write)              RECALL (four-way retrieval)     │
│  ┌─────────────────┐              ┌──────────────────────────┐    │
│  │ text → embed     │              │ Semantic (sqlite-vec)    │    │
│  │ → SQLite chunk   │              │ Keyword  (FTS5 BM25)    │    │
│  │ → queue extract  │              │ Graph    (entity walk)   │    │
│  └────────┬────────┘              │ Temporal (date filter)   │    │
│           │                       └────────────┬─────────────┘    │
│           ▼                                    ▼                  │
│  EXTRACT (background)             RRF + Trust Weighting           │
│  ┌─────────────────┐              ┌──────────────────────────┐    │
│  │ Ollama → entities│              │ Ranked results +         │    │
│  │ → relations      │              │ opinions + observations  │    │
│  │ → chunk_entities │              └──────────────────────────┘    │
│  └─────────────────┘                                              │
│                                                                   │
│  REFLECT (scheduled learning)                                     │
│  ┌──────────────────────────────────────────────────────────┐     │
│  │ unreflected facts → Ollama → observations + opinions     │     │
│  │ (batch, not real-time — avoids latency on every write)   │     │
│  └──────────────────────────────────────────────────────────┘     │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐     │
│  │               <agentId>.engram (SQLite)                   │     │
│  │  chunks | entities | relations | chunk_entities           │     │
│  │  opinions | observations | reflect_log | belief_journal   │     │
│  │  suggestions | suggestion_journal                         │     │
│  │  extraction_queue | bank_config | chunks_fts (FTS5)       │     │
│  │  working_memory                                           │     │
│  └──────────────────────────────────────────────────────────┘     │
└───────────────────────────────────────────────────────────────────┘
```

## The Engram Pipeline (Biological Metaphor)

The system mirrors how biological memory works:

1. **Encoding** (retain) — Sensory input is rapidly stored as a raw trace. Fast, no deliberation. Entity extraction happens in the background, the way the hippocampus processes memories during idle time.

2. **Consolidation** (extract) — Background process strengthens the trace by linking it to existing knowledge structures. Entities are resolved, relationships mapped, the knowledge graph densifies.

3. **Reconsolidation** (reflect) — Periodic review of accumulated traces produces higher-order understanding. Observations emerge from patterns. Beliefs form and update with confidence. Old observations get refined as new evidence arrives. This is the learning loop.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tom-swift-tech/engram](https://github.com/tom-swift-tech/engram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
