---
trigger: always_on
description: **BLOCKING REQUIREMENT**: Before using Read, Glob, Grep, or Agent/Explore to access this codebase, you MUST first call at least one Noumenon MCP tool (`noumenon_status`, `noumenon_query`, or `noumenon_ask`). This is enforced by a PreToolUse hook — file-reading tools will be blocked until a Noumenon MCP query has been made.
---

# CRITICAL: Use Noumenon MCP Tools First

**BLOCKING REQUIREMENT**: Before using Read, Glob, Grep, or Agent/Explore to access this codebase, you MUST first call at least one Noumenon MCP tool (`noumenon_status`, `noumenon_query`, or `noumenon_ask`). This is enforced by a PreToolUse hook — file-reading tools will be blocked until a Noumenon MCP query has been made.

**DO NOT skip this. DO NOT fall back to reading files without querying the knowledge graph first.**

## Why

This project has its own MCP server (`noumenon`) that provides a pre-compiled knowledge graph of the codebase. It knows about file structure, namespaces, dependencies, commit history, code complexity, architectural layers, and more. Querying the knowledge graph is faster, cheaper, and more accurate than scanning raw files.

## Required Workflow

1. **First** — Call `noumenon_status` to confirm the repo is imported.
2. **Then discover** — Use `noumenon_query` for structured questions, or `noumenon_ask` for open-ended exploration.
3. **Then read files** — Only after the knowledge graph has told you *which* files matter and *why*, read them with Read for specific details.

## When to Use Each Tool

- **Before modifying a file** — Use `noumenon_query` or `noumenon_ask` to understand the file's role, its relationships, and recent change history.
- **When searching for code** — Use `noumenon_query` instead of Glob/Grep. The graph knows about file structure, namespaces, dependencies, and commit history.
- **When exploring the codebase** — Use `noumenon_get_schema` to see what's in the graph, `noumenon_list_queries` to see available queries, and `noumenon_ask` for open-ended questions.
- **When understanding impact** — Before a refactor or rename, query the knowledge graph for what depends on the thing you're changing.

## Available Tools

| Tool | Purpose |
|------|---------|
| `noumenon_status` | Check entity counts — verify the repo has been imported |
| `noumenon_get_schema` | See all attributes and types in the knowledge graph |
| `noumenon_list_queries` | List available named Datalog queries |
| `noumenon_query` | Run a named Datalog query (some require params — check `list_queries` first) |
| `noumenon_ask` | Ask a natural-language question — AI-powered iterative querying |
| `noumenon_import` | Import git history and file structure (idempotent — safe to re-run) |
| `noumenon_update` | Sync knowledge graph with latest git state (import + enrich; pass `analyze=true` for LLM analysis) |
| `noumenon_analyze` | Run LLM analysis on files — by default only unanalyzed; pass `reanalyze` for re-analysis (all, prompt-changed, model-changed, stale) |
| `noumenon_enrich` | Extract cross-file import/dependency graph deterministically (no LLM calls) |
| `noumenon_synthesize` | Identify architectural components from analyzed codebase data (macro analysis) |
| `noumenon_list_databases` | List all noumenon databases with entity counts, pipeline stages, and cost |
| `noumenon_digest` | Run the full pipeline: import, enrich, analyze, synthesize, benchmark (each step idempotent, skippable) |
| `noumenon_benchmark_run` | Run benchmark comparing LLM answers across knowledge graph layers (expensive) |
| `noumenon_benchmark_results` | Get benchmark results (latest run or by ID) |
| `noumenon_benchmark_compare` | Compare two benchmark runs by score differences per layer |

After querying the knowledge graph, use Read/Glob/Grep surgically on specific files for details the graph doesn't contain (exact implementation, line-level edits). The MCP gives you direction; file tools give you precision.

# Project Instructions

## Security

Never read, write, display, or reference the contents of `.env` or `.env.local` files. These contain secrets. Use `.env.example` for documentation only.

## Clojure Style

### Design

- **Data-oriented**: Prefer plain maps and vectors over custom types. Data > functions > macros.
- **Pure core, impure shell**: Pure transformation functions in the core, side effects (I/O, Datomic transacts, HTTP) at the edges.
- **Minimal surface area**: Small functions that compose. No speculative abstractions.
- **EDN for configuration**: Queries, schema, prompts — all EDN resources, not code.
- **Functions under 10 LOC**: Most functions should be under 5 lines. If a function is long, break it up.
- **Max 3–4 positional params**: Beyond that, use an options map.
- **No deep nesting**: Deeply nested `if`/`cond`/`when` blocks are a sign a function is doing too much. Extract named helpers or use `cond`/`condp` to flatten decision trees. If a threading pipeline exceeds ~5–6 steps, break it into named intermediate values or extract a helper.

### Point-free / tacit style

Prefer point-free composition over naming intermediate arguments. The pipeline *is* the documentation.

```clojure
;; good — point-free pipeline, reads top-down
(->> (d/q query db)
     (remove (comp #{"db" "fressian"} namespace first))
     (sort-by first)
     (mapv make-attr-map))

;; avoid — unnecessary lambda wrapping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leifericf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
