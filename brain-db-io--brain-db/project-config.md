---
trigger: always_on
description: This file is loaded automatically by Claude Code at the start of every session. It tells Claude what this project is, what's authoritative, and how to operate.
---

# Brain — Claude Code Project Context

This file is loaded automatically by Claude Code at the start of every session. It tells Claude what this project is, what's authoritative, and how to operate.

**For autonomous-mode operating rules** (`claude --dangerously-skip-permissions`), see [`AUTONOMY.md`](AUTONOMY.md). That file defines the execution loop, hard rules, and stop conditions. Read it before doing any work.

---

## 1. What this project is

**Brain** is a cognitive substrate for AI agents — a database where the primitives are cognitive operations (encode, recall, plan, reason, forget) instead of tables/documents/vectors.

v1.0 ships in two layers:

- **Substrate** (spec §00–§16, phases 0–14): vector memory store. WAL, HNSW, wire protocol, cognitive primitives (ENCODE/RECALL/PLAN/REASON/FORGET), HTTP/WS/SSE, observability.
- **Knowledge layer** (spec §17–§31, phases 15–24): typed entities, statements (Fact/Preference/Event), relations, schema DSL, three-tier extractors (pattern → classifier → LLM), hybrid retrieval (semantic + lexical + graph with RRF fusion).

The knowledge layer activates only when a schema is declared via `SCHEMA_UPLOAD`. A deployment that never declares a schema runs as a pure vector substrate (substrate-only mode) — a first-class deployment posture, not a legacy mode. The `v1.0.0` tag lands at the end of Phase 24, after the combined acceptance suite passes.

We are **building the implementation**. The design is already done.

## 2. Single source of truth

The `spec/` directory is **authoritative**. 32 specification sections — 17 substrate (§00–§16) + 15 knowledge layer (§17–§31).

- **The spec is read-only.** Don't edit it. Spec changes go through the user.
- **The spec wins.** If code disagrees with spec, fix the code.
- **The spec is comprehensive.** If a question seems unanswered, look harder before deciding it's missing.

Quick-find:

| Question | Spec file |
|---|---|
| What does ENCODE do? | `spec/09_cognitive_operations/02_encode.md` |
| What does RECALL return? | `spec/09_cognitive_operations/03_recall.md` |
| What's the wire frame format? | `spec/03_wire_protocol/03_frame_header.md` |
| How does WAL recovery work? | `spec/05_storage_arena_wal/08_recovery.md` |
| What's the redb schema? | `spec/07_metadata_graph/02_table_layout.md` |
| Why HNSW M=16? | `spec/06_ann_index/02_parameters.md` |
| What error codes exist? | `spec/03_wire_protocol/10_errors.md` |
| What's the latency target? | `spec/16_benchmarks_acceptance/02_latency_targets.md` |
| What's the three-layer model? | `spec/17_knowledge_model/00_purpose.md` |
| How does entity resolution work? | `spec/18_entities/01_resolution.md` |
| Fact vs Preference vs Event rules? | `spec/17_knowledge_model/02_three_statement_kinds.md` |
| What does the schema DSL look like? | `spec/21_schema_dsl/01_grammar.md` |
| How do the three extractor tiers compose? | `spec/22_extractors/00_purpose.md` |
| How does RRF fusion work? | `spec/23_retrievers/01_rrf_fusion.md` + `spec/24_hybrid_query/00_purpose.md` |
| What knowledge-layer wire opcodes exist? | `spec/28_knowledge_wire_protocol/00_purpose.md` |
| What's the knowledge-layer storage layout? | `spec/26_knowledge_storage/00_purpose.md` |
| What's the combined acceptance gate? | `spec/31_complete_acceptance/00_purpose.md` |

The full directory map is in [`spec/00_master_overview/02_doc_map.md`](spec/00_master_overview/02_doc_map.md).

## 3. How work is structured

The roadmap is in three layers:

1. **[`ROADMAP.md`](ROADMAP.md)** — high-level phase index. One-line per phase.
2. **[`docs/development/phases/phase-NN-*.md`](docs/development/phases/)** — detailed sub-task breakdown for each phase. Reads, writes, "done when" criteria.
3. **[`AUTONOMY.md`](AUTONOMY.md)** — operating rules (commit format, stop conditions, scope guards).

When asked "what's next?", the answer is always: the lowest-numbered unfinished sub-task in the active phase doc.

The `/next-task` slash command finds it for you.

## 4. Architecture in one paragraph

Linux server. Connection layer (Tokio) accepts TCP; each request dispatches to one of N **shards**. Each shard runs a **Glommio** executor (thread-per-core, io_uring) and owns its data: a memory-mapped **arena** for vectors, a **WAL** with O_DIRECT + `pwritev2(RWF_DSYNC)` group commit, a **redb** B-tree for metadata, an **HNSW** index in RAM. **Single-writer-per-shard**: writes don't lock; reads use **ArcSwap** + **crossbeam-epoch**. Twelve **background workers** handle decay, consolidation, HNSW maintenance, GC, etc. The substrate **owns the embedding model** (BGE-small via candle, 384-dim); clients send text.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brain-db-io/brain-db](https://github.com/brain-db-io/brain-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
