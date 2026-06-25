---
trigger: always_on
description: This file is for agents connected to memoryweb via MCP. It tells you how to use
---

# AGENTS.md — guidance for AI agents using memoryweb

This file is for agents connected to memoryweb via MCP. It tells you how to use
the tools correctly, what the tools will and won't surface, and how to behave
around archiving, drift, and knowledge gaps.

---

## What memoryweb is

A persistent knowledge graph for a project or set of projects. You file concepts
and decisions as nodes, connect them with typed narrative edges, and retrieve
them by searching or following connections.

It is called **memoryweb**. Nothing else.

---

## Core tool guide

### Orientation at session start

Always call `orient` for the relevant domain before answering questions about a
project. Do not rely on your context window or training for project state.

`orient` returns all live nodes for the domain, recent activity, and a
`declared_spine` — the curated history of key decisions in chronological order.
Weigh the spine heavily when synthesising.

If you do not know what domains exist, call `domains` first.

### Filing knowledge

- `remember` — file a concept, decision, or finding. Accepts `related_to` to
  auto-connect at creation. Returns `suggested_connections` and
  `possible_duplicates` — act on these before filing more nodes.
  Supply an `items` array to file multiple nodes in one transaction.
- `revise` — update `label`, `description`, `why_matters`, `tags`, or
  `occurred_at` on a live node without archiving it. Supply an `items` array for
  batch updates. Writes an audit log entry on every call.

**Before filing a node**: search first. If a similar node exists, suggest
linking with `connect` rather than creating a duplicate. Unfiled duplicates are
the primary cause of orphan nodes and audit drift.

**The `why_matters` field is the most important one** — it is what makes a node
retrievable from oblique search terms. Never skip it.

**ALWAYS call `connect` for any `suggested_connections` before ending your
session.** A node with no connections is nearly worthless.

### Connecting memories

- `connect` — connect two nodes with a typed relationship and narrative *because*.
  Both nodes must exist first. Supply an `items` array to create multiple
  connections in one transaction.
- `disconnect` — remove a connection by edge ID. Hard delete — obtain the ID
  from `recall`.
- `suggest_connections` — read-only; returns up to 5 candidate connections from
  the same domain for a given node. Each suggestion includes a `domain` field so
  you can scope a cross-domain connect call correctly.

### Retrieving memories

- `recall` — retrieve a node and all its connections by ID.
- `search` — text and semantic search across `label`, `description`,
  `why_matters`, and `tags`. Returns `truncated: true` when results hit the
  limit. Use words that appear in stored content — not conceptual paraphrases.
  When Ollama is running, results include a `semantic_distance` field.
- `recent` — what was filed recently. Set `group_by_domain=true` (with no
  domain) to see activity broken down per domain.
- `history` — nodes ordered by when they actually occurred. Supports `from`/`to`
  date range filtering, `tags` filtering, and `important_only` for curated spine
  entries only.
- `why_connected` — find the reasoning linking two named concepts (by label).
- `trace` — shortest path between two nodes by ID, up to 6 hops. Returns
  intermediate nodes and edges. Synthesise the result as a narrative.
- `orient` — full domain summary: all nodes, recent activity, and the declared
  spine. Includes `total_nodes` and `server_version`.
- `visualise` — Mermaid flowchart for a domain or a single node's neighbourhood
  (pass `memory_id`). Always output the mermaid string inside a mermaid code block.
- `significance` — dual-signal importance analysis for a domain. Returns four
  sections: `declared` (nodes with `occurred_at` set), `structural` (ranked by
  recency-weighted inbound degree), `uncurated` (structural top-N without
  `occurred_at` — curation candidates), and `potentially_stale` (declared but
  low structural score). The gap between uncurated and potentially_stale is the
  most actionable output.

**All retrieval tools only return live nodes.** Archived nodes are invisible.
If something seems to be missing, call `audit(mode=archived)` to check whether
it was archived, or `audit(mode=stale)` to surface drift candidates.

### Archiving and maintenance

- `forget` — archive a single node with a reason. Follow the forget protocol
  below.
- `forget_all` — archive multiple nodes atomically in a single call.
- `restore` — restore an archived node so it surfaces in search again.
- `audit` — surface nodes that need attention. Three modes:
  - `mode=stale` — stale, contradicted, duplicated, or overdue transient nodes
  - `mode=orphans` — live nodes with zero connections
  - `mode=archived` — review what has been archived

### Domain management

- `domains` — list all domains with at least one live node, and all registered
  aliases.
- `alias` — manage domain aliases. Actions: `add`, `remove`, `resolve`, `list`.
  Register short aliases so both `dg` and `deep-game` return the same results.
- `rename_domain` — rename a domain in place. Automatically registers an alias

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [corbym/memoryweb](https://github.com/corbym/memoryweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
