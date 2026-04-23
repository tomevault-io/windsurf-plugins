---
trigger: always_on
description: This repository is built with Codex using parallel agents. Treat this file as the authoritative operating guide for any agent working in the repo.
---

# AGENTS.md

This repository is built with Codex using parallel agents. Treat this file as the authoritative operating guide for any agent working in the repo.

## Product summary

`chum-mem` is a cloud-native persistent memory system for coding agents. It supports Claude, Codex, and Gemini clients and uses PostgreSQL, pgvector, and Chroma-backed hybrid retrieval. The current architecture is the v2.2.3 PCKC runtime:

- claims are the unit of memory
- proof is the unit of trust
- compiled minimal proof sets are the unit of context
- repository and session graphs are first-class retrieval surfaces
- retrieval is hybrid: lexical + pgvector + Chroma

The system supports organizations, teams, members, projects, and personal API tokens used by local clients to connect to the MCP server.

## Primary goals

- implement a secure multi-tenant backend
- support provider-normalized ingestion and retrieval
- provide an MCP-first memory service with optional admin surfaces
- make the answering model more reliable as knowledge grows, not less reliable
- preserve reproducible agent work and low-friction handoff

## Source of truth

Use these files first:

1. `docs/INSTRUCTION.md`
2. `docs/ARCHITECTURE_SPEC.md`
3. this `AGENTS.md`
4. `docs/research/v2.2.3-pckc/README.md`
5. `docs/research/v2.2.2-pckc/README.md`
6. `docs/research/v2.2.2-pckc/results/COMPARISON.md`
7. relevant skill in `.codex/skills/`
8. relevant prompt in `.codex/prompts/`

If research docs, benchmarks, and runtime code disagree, prefer:

1. architecture spec
2. current runtime code
3. benchmark artifacts
4. older research notes

When you discover a real mismatch, update docs in the same task or leave a precise follow-up note.

## Working rules

- make focused changes with clear scope
- do not rewrite architecture without updating the spec
- preserve tenant isolation in all data-access code
- prefer explicit types and runtime validation
- add tests for any non-trivial behavior
- do not store plaintext API tokens
- do not trust caller-supplied tenant identifiers
- keep provider-specific logic behind adapters
- preserve provenance for memory derivation and retrieval
- preserve authority and verification metadata on claims
- preserve supersession and contradiction semantics
- do not replace proof-carrying claims with narrative summaries

## Default memory gate

For every new code task, use `chum-memory` first.

Required flow:

1. call `knowledge_query(search, layer:"repository")` and `mem_search` in parallel before reading or grepping
2. keep `mem_search` compact: `mode=hybrid`, `disclosureLevel=overview`, small `limit`
3. select relevant IDs from results
4. call `memory_get_batch` only for selected IDs
5. call `context_build` or `context_compile_v2` only when a compact proof set is actually needed

Do not skip this flow unless the user explicitly asks to continue without memory.

Repository questions should default to repository truth. Session memory is a secondary witness for decisions, tasks, bugs, and continuity.

## PCKC expectations

v2.2.3 is not generic summary memory. It is a proof-carrying claim system with deterministic governance.

Key operational rules:

- prefer typed claims over prose summaries
- prefer `verified`, `user_confirmed`, `tool_verified`, `test_verified`, and repository-derived truth
- treat `model_derived` and `unverified` claims as non-durable unless explicitly justified by the runtime contract
- never cite superseded claims as current truth
- surface conflicts explicitly; do not silently average contradictory claims
- use `context_compile_v2` when hard budget discipline and proof-gap signaling matter
- use `knowledge_report` and `knowledge_query` for graph-native reasoning before falling back to grep
- governance states (active/pinned/archived/rejected) are deterministic operator controls; respect them in retrieval
- pinned claims are elevated in ranking; archived/rejected claims are filtered from default search results

## Current architecture snapshot

The applied v2.2.3 architecture includes:

- typed claim extraction with authority and verification metadata
- belief gate enforcement during derivation
- typed embedding partitions for claim-type-aware retrieval
- deep repository graph with containment edges and cross-file call resolution
- session graph construction with restored inter-claim edges
- hierarchical Leiden communities with `level` and `community_path`
- project-scoped community caching for graph-heavy queries
- hard-budget minimal-proof compilation via `context_compile_v2`
- continuation-aware ranking with `is_continuation` flag and actionable-claim boosting
- section-aware context assembly with baseline queries for all 6 core section types
- deterministic memory governance (active/pinned/archived/rejected) with audit history
- governance-aware scoring in ranking and SQL-level filtering
- cross-layer unified reporting via JSON response with `crossLayerSummary`

Known active gaps from the latest benchmark/docs:

- repository-only context fill rate remains at 0.125 (needs repository-derived claims or scaffolding)
- continuation boost magnitudes are analytically tuned, not yet benchmark-validated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sly-codechum/chum-mem](https://github.com/sly-codechum/chum-mem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
