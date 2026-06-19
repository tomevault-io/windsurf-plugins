---
trigger: always_on
description: Use when persisting decisions, preferences, or architectural facts across coding sessions, or querying what was previously decided, built, or constrained in this project.
---


# Temporal Reasoning

Perfect memory. Exact reasoning. Complete history.

Temporal Reasoning gives AI coding agents bi-temporal graph memory: query any past state, traverse live dependency graphs, and correlate architectural decisions with structural change — all with deterministic Datalog, no fuzzy retrieval.

## The Core Idea

Every session starts from zero — you ask questions already answered, write code that contradicts decisions already made, and miss constraints established weeks ago. Temporal Reasoning fixes this: a persistent bi-temporal graph store you write to and query at any time, so context survives across sessions.

**The two habits this skill builds:**
- **Write immediately** when the user establishes something worth keeping (decision, preference, constraint)
- **Read before acting** when the user asks about the past, or when you're about to modify something where past decisions might apply

## When to Write (vulcan_transact)

Write to memory when the user's words signal a durable fact:

| Signal | Examples | What to store |
|--------|----------|--------------|
| Decision language | "we'll use X", "going with Y", "we decided Z" | The decision + what was rejected |
| Preference | "I prefer", "I don't like", "always/never use" | The preference + why (if given) |
| Constraint | "must be", "can't use", "prioritize X over Y" | The constraint + the tradeoff |
| Dependency | "depends on", "requires", "calls into" | The relationship |
| Architecture | system structure, component roles, data flows | The structure + rationale |

Store the *why* when you have it — a reason like "chosen for async support" is far more useful than the bare fact "using FastAPI".

After every write, say: "I've stored that in memory." and summarize what was stored.

## When to Read (vulcan_query)

Query memory before you answer or act, when:
- The user asks about past decisions, architecture, preferences, or constraints
- The user says "what did we...", "how did we...", "why did we...", "what was our..."
- The user references something from "earlier", "before", "last time"
- You're about to write code that touches existing architecture
- There's any ambiguity about what was established before

Say "Let me check memory..." before querying. Then:
- If memory has relevant facts → cite them specifically and ground your answer in them
- If memory is empty or returns nothing relevant → say "Memory doesn't have anything recorded about this" and ask if they'd like to share context you can store

**Query first, answer second.** The reason: a confident answer that contradicts a stored decision is far more damaging than taking a moment to check.

## When to Retract (vulcan_retract)

Retract when:
- The user explicitly says "remove", "delete", "retract", "forget", "that's no longer true"
- A fact has been superseded by a newer decision
- A fact was stored incorrectly

After retraction, say: "I've removed that from memory (the original is preserved in history)."

## What NOT to Store

Skip transient observations, intermediate reasoning, raw code snippets, and restatements of what the user just said. Store durable, cross-session facts only: decisions, preferences, constraints, dependencies, architecture.

## Entity Idents and Attribute Names

Facts are stored as triples: `[entity attribute value]`. The entity ident is the organizing key — it carries all the identity and namespacing you need. Use flat, descriptive attribute names.

**Entity idents** should be meaningful and namespaced: `:decision/postgres`, `:preference/no-db-mocks`, `:constraint/python-version`

**Attribute names** should be flat and self-explanatory: `:description`, `:rationale`, `:date`, `:alias`, `:entity-type`, `:calls`, `:depends-on`, `:motivated-by`, `:governs`

```
[:decision/postgres :description "PostgreSQL 15 — primary database"]
[:decision/postgres :rationale "ACID compliance + JSON support; tradeoff: lower write throughput"]
[:preference/no-db-mocks :description "always use real DB connections in tests"]
[:preference/no-db-mocks :rationale "mock/prod divergence caused silent migration failure"]
```

To retrieve all facts for an entity, query by ident directly — no need to know attribute names in advance:
```python
query("[:find ?a ?v :where [:decision/postgres ?a ?v]]")
```

Before adding new facts about an entity, query it first to find existing attributes and avoid duplication.

## Entity Resolution

Before storing a new entity, always check for existing canonical idents and aliases:

```datalog
[:find ?e ?desc :where [?e :description ?desc]]
[:find ?e ?a :where [?e :alias ?a]]
```

If a reference matches an existing ident or alias, reuse that exact ident.
Only mint a new ident if the entity is genuinely new.

Canonical ident form: lowercase, hyphens only — `:decision/redis` not `:decision/Redis_cache`.

Allowed entity types: `:decision/`, `:preference/`, `:constraint/`, `:dependency/`, `:module/`, `:function/`, `:class/` (code structure — auto-ingested); `:commit/`, `:tag/`, `:ingestion/` are system-only (written by `vulcan_ingest_git`), do not write to them directly
Required attribute on all types: `:description`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [project-minigraf/temporal_reasoning](https://github.com/project-minigraf/temporal_reasoning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
