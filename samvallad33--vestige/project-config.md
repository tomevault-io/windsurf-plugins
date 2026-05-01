---
trigger: always_on
description: Vestige is your long-term memory AND reasoning engine. 29 stateful cognitive modules implement real neuroscience: FSRS-6 spaced repetition, synaptic tagging, prediction error gating, hippocampal indexing, spreading activation, reconsolidation, and dual-strength memory theory. **Use it automatically. Use it aggressively.**
---

# Vestige v2.0.4 — Cognitive Memory & Reasoning System

Vestige is your long-term memory AND reasoning engine. 29 stateful cognitive modules implement real neuroscience: FSRS-6 spaced repetition, synaptic tagging, prediction error gating, hippocampal indexing, spreading activation, reconsolidation, and dual-strength memory theory. **Use it automatically. Use it aggressively.**

**NEW: `deep_reference` — call this for ALL factual questions.** It doesn't just retrieve — it REASONS across memories with FSRS-6 trust scoring, intent classification, contradiction analysis, and generates a pre-built reasoning chain. Read the `reasoning` field FIRST.

---

## Session Start Protocol

Every conversation, before responding to the user:

```
session_context({
  queries: ["user preferences", "[current project] context"],
  context: { codebase: "[project]", topics: ["[current topics]"] },
  token_budget: 2000
})
```

Then check `automationTriggers` from response:
- `needsDream` → call `dream` (consolidates memories, discovers hidden connections)
- `needsBackup` → call `backup`
- `needsGc` → call `gc(dry_run: true)` then review
- totalMemories > 700 → call `find_duplicates`

Say "Remembering..." then retrieve context before answering.

> **Fallback:** If `session_context` unavailable: `search` × 2 → `intention` check → `system_status` → `predict`.

---

## Complete Tool Reference (23 Tools)

### session_context — One-Call Initialization
```
session_context({
  queries: ["user preferences", "project context"],  // search queries
  context: { codebase: "project-name", topics: ["svelte", "rust"], file: "src/main.rs" },
  token_budget: 2000,        // 100-100000, controls response size
  include_status: true,       // system health
  include_intentions: true,   // triggered reminders
  include_predictions: true   // proactive memory predictions
})
```
Returns: markdown context + `automationTriggers` + `expandable` IDs for on-demand retrieval.

### smart_ingest — Save Anything
**Single mode** — auto-decides CREATE/UPDATE/SUPERSEDE via Prediction Error Gating:
```
smart_ingest({
  content: "What to remember",
  tags: ["tag1", "tag2"],
  node_type: "fact",          // fact|concept|event|person|place|note|pattern|decision
  source: "optional reference",
  forceCreate: false          // bypass dedup when needed
})
```
**Batch mode** — save up to 20 items in one call (session end, pre-compaction):
```
smart_ingest({
  items: [
    { content: "Item 1", tags: ["session-end"], node_type: "fact" },
    { content: "Item 2", tags: ["bug-fix"], node_type: "fact" }
  ]
})
```
Each item runs the full cognitive pipeline: importance scoring → intent detection → synaptic tagging → hippocampal indexing → PE gating → cross-project recording.

### search — 7-Stage Cognitive Search
```
search({
  query: "search query",
  limit: 10,                  // 1-100
  min_retention: 0.0,         // filter by retention strength
  min_similarity: 0.5,        // minimum cosine similarity
  detail_level: "summary",    // brief|summary|full
  context_topics: ["rust", "debugging"],  // boost topic-matching memories
  token_budget: 3000,         // 100-100000, truncate to fit
  retrieval_mode: "balanced"  // precise|balanced|exhaustive (v2.1)
})
```
Retrieval modes: `precise` (fast, no activation/competition), `balanced` (default 7-stage pipeline), `exhaustive` (5x overfetch, deep graph traversal, no competition suppression).

Pipeline: Overfetch → Rerank (cross-encoder) → Temporal boost → Accessibility filter (FSRS-6) → Context match (Tulving 1973) → Competition (Anderson 1994) → Spreading activation. **Every search strengthens the memories it finds (Testing Effect).**

### memory — Read, Edit, Delete, Promote, Demote
```
memory({ action: "get", id: "uuid" })           // full node with all FSRS state
memory({ action: "edit", id: "uuid", content: "updated text" })  // preserves FSRS state, regenerates embedding
memory({ action: "delete", id: "uuid" })
memory({ action: "promote", id: "uuid", reason: "was helpful" })  // +0.20 retrieval, +0.10 retention, 1.5x stability
memory({ action: "demote", id: "uuid", reason: "was wrong" })     // -0.30 retrieval, -0.15 retention, 0.5x stability
memory({ action: "state", id: "uuid" })          // Active/Dormant/Silent/Unavailable + accessibility score
memory({ action: "get_batch", ids: ["uuid1", "uuid2", "uuid3"] })  // retrieve up to 20 full memories at once (v2.1)
```
Promote/demote does NOT delete — it adjusts ranking. Demoted memories rank lower; alternatives surface instead.
`get_batch` is designed for batch retrieval of expandable overflow IDs from search/session_context.

### codebase — Code Patterns & Architectural Decisions
```
codebase({ action: "remember_pattern", name: "Pattern Name",
  description: "How it works and when to use it",
  files: ["src/file.rs"], codebase: "project-name" })

codebase({ action: "remember_decision", decision: "What was decided",
  rationale: "Why", alternatives: ["Option A", "Option B"],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samvallad33/vestige](https://github.com/samvallad33/vestige) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
