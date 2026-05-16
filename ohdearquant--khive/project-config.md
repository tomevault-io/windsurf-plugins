---
trigger: always_on
description: This file is for AI agents (and the humans configuring them) using khive as the research runtime.
---

# khive — Agent Usage Guide

This file is for AI agents (and the humans configuring them) using khive as the research runtime.

khive gives your agent three things:

1. **A knowledge graph** — typed entities + edges you build as you work
2. **Notes** — observations, insights, questions, decisions, references that persist across sessions
3. **Pattern matching queries** — GQL/SPARQL traverse over the graph

If you're working on khive itself (writing code in this repo), see `CLAUDE.md` instead.

---

## Core verbs

All verbs are available via MCP ([ADR-023](docs/adr/ADR-023-verb-consolidated-mcp-surface.md)).

| Verb        | What it does                                     | When to use                                              |
| ----------- | ------------------------------------------------ | -------------------------------------------------------- |
| `create`    | Add an entity or note                            | New concept, paper, observation, decision worth tracking |
| `get`       | Fetch any record by UUID (auto-detects type)     | When you have a UUID and need the full record            |
| `search`    | Text + semantic search over entities or notes    | Finding things by content similarity                     |
| `list`      | Structured filtering (by kind, tags, etc.)       | Browsing a category or namespace                         |
| `update`    | Patch properties, tags, or content (by UUID)     | Correcting or enriching an existing record               |
| `delete`    | Soft-delete (or hard-delete) a record (by UUID)  | Removing stale or incorrect data                         |
| `link`      | Connect two nodes with a typed relation          | When relationships emerge from research                  |
| `traverse`  | Multi-hop graph walk with depth/relation filters | Structural context — lineages, paths, clusters           |
| `neighbors` | Immediate neighbors of a node                    | "What connects to this entity?"                          |
| `query`     | GQL/SPARQL query string → SQL                    | Complex pattern matching over the graph                  |
| `merge`     | Deduplicate two entities into one (v0.1)         | "LoRA" and "Low-Rank Adaptation" are the same concept    |

**11 tools in v0.1.** `get`, `update`, `delete` are UUID-only — they auto-detect whether the
record is an entity, note, or edge. `create`, `list`, `search` require `kind=entity|note` (or
`kind=edge` for `list`).

**Deferred (not available in v0.1):** `supersede` (use `link(..., relation="supersedes")` as a
workaround), `request` (batch DSL), note merge (only entity merge is implemented).

### Notes vs entities

- **Entities** = things in the world: concepts, papers, people, projects, datasets, orgs.
  Graph nodes with typed edges between them.
- **Notes** = your observations about the world: what you noticed, concluded, decided, asked, cited.
  Temporal records with salience and optional graph edges (via `annotates`).

Use `create(kind="entity", entity_kind="concept", ...)` for entities.
Use `create(kind="note", note_kind="observation", ...)` for notes.

---

## The 6 entity kinds (closed set — [ADR-001](docs/adr/ADR-001-entity-kind-taxonomy.md))

| Kind       | What it represents                                      |
| ---------- | ------------------------------------------------------- |
| `concept`  | Algorithms, techniques, architectures, theories, models |
| `document` | Papers, preprints, technical reports, blog posts, books |
| `dataset`  | Benchmarks, corpora, evaluation sets                    |
| `project`  | Codebases, libraries, tools, frameworks                 |
| `person`   | Researchers, engineers, authors                         |
| `org`      | Labs, companies, institutions                           |

`concept` is the default. Use `properties` for finer distinctions (`type: "paper"`,
`domain: "attention"`, `status: "implemented"`).

---

## The 5 note kinds (closed set — [ADR-019](docs/adr/ADR-019-note-kind-taxonomy.md))

| Kind          | What it records                               |
| ------------- | --------------------------------------------- |
| `observation` | An empirical capture — what you noticed       |
| `insight`     | A synthetic conclusion from observations      |
| `question`    | An open inquiry or research direction         |
| `decision`    | A committed choice with rationale             |
| `reference`   | An external pointer with context (paper, URL) |

`observation` is the default. Notes can annotate entities via `create(kind="note",
annotates=[entity_id], ...)`.

---

## The 13-relation ontology (closed set — [ADR-002](docs/adr/ADR-002-edge-ontology.md))

When you `link` nodes, use ONLY these relations:

### Structure

- `contains` — parent → child (system contains module)
- `part_of` — inverse of contains
- `instance_of` — specific is a case of general

### Derivation

- `extends` — child builds on parent (Flash Causal extends Flash Tiled)
- `variant_of` — A is a modified version of B (QLoRA variant_of LoRA)
- `introduced_by` — concept first described in paper/by person
- `supersedes` — new replaces old entirely

### Dependency

- `depends_on` — consumer needs dependency at runtime/build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohdearquant/khive](https://github.com/ohdearquant/khive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
