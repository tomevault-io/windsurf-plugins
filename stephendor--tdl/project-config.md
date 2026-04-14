---
trigger: always_on
description: **MANDATORY: use `run_pipeline` — do NOT grep, glob, or read files manually.**
---

## vexp context tools <!-- vexp v1.3.11 -->

**MANDATORY: use `run_pipeline` — do NOT grep, glob, or read files manually.**
vexp returns pre-indexed, graph-ranked context in a single call.

### Workflow
1. `run_pipeline` with your task description — ALWAYS FIRST (replaces all other tools)
2. Make targeted changes based on the context returned
3. `run_pipeline` again only if you need more context

### Available MCP tools
- `run_pipeline` — **PRIMARY TOOL**. Runs capsule + impact + memory in 1 call.
  Auto-detects intent. Includes file content. Example: `run_pipeline({ "task": "fix auth bug" })`
- `get_context_capsule` — lightweight, for simple questions only
- `get_impact_graph` — impact analysis of a specific symbol
- `search_logic_flow` — execution paths between functions
- `get_skeleton` — compact file structure
- `index_status` — indexing status
- `get_session_context` — recall observations from sessions
- `search_memory` — cross-session search
- `save_observation` — persist insights (prefer run_pipeline's observation param)

### Agentic search
- Do NOT use built-in file search, grep, or codebase indexing — always call `run_pipeline` first
- If you spawn sub-agents or background tasks, pass them the context from `run_pipeline`
  rather than letting them search the codebase independently

### Smart Features
Intent auto-detection, hybrid ranking, session memory, auto-expanding budget.

### Multi-Repo
`run_pipeline` auto-queries all indexed repos. Use `repos: ["alias"]` to scope. Run `index_status` to see aliases.
<!-- /vexp -->

---

## Vault-Engine MCP Server

On-device access to the Obsidian research vaults. Uses QMD (local embeddings)
for hybrid search and a wikilink graph index for link-aware context expansion.

### Available tools

| Tool             | Purpose                                           | When to use                                             |
| ---------------- | ------------------------------------------------- | ------------------------------------------------------- |
| `vault_query`    | Graph-augmented search across vaults              | Finding methodology, decisions, literature, conventions |
| `vault_get`      | Read a specific page with backlinks/forward links | Loading CONVENTIONS, \_project.md, Computational-Log    |
| `vault_graph`    | N-hop wikilink neighborhood                       | Understanding how concepts connect                      |
| `vault_skeleton` | Token-efficient page summaries (80-90% reduction) | Scanning multiple pages without context overflow        |
| `vault_status`   | Dashboard with health metrics                     | Session start, checking paper pipeline status           |
| `cross_vault`    | Shared concepts between TDA & Counting Lives      | Finding Two Lenses connections                          |
| `vault_observe`  | Save observations linked to vault pages           | Recording insights for cross-session memory             |

### Vault identifiers

- `tda` — TDA-Research vault (methodology, papers, literature)
- `cl` — Counting Lives vault (book manuscript, sources, research)

### Workflow integration

1. **Session start:** Call `vault_get("CONVENTIONS", vault="tda")` to load locked rules
2. **Paper work:** Call `vault_get("03-Papers/P01-A/_project.md")` before writing
3. **Methodology questions:** Call `vault_query("your question")` — expands via wikilinks
4. **After decisions:** Call `vault_observe("decision text", page="CONVENTIONS")`

### Key vault pages to know

| Page                   | Purpose                                                   |
| ---------------------- | --------------------------------------------------------- |
| `CONVENTIONS`          | Locked always/never rules — **check before implementing** |
| `Computational-Log`    | All logged results and decisions                          |
| `Pipeline-Overview`    | Pipeline architecture                                     |
| `markov-memory-ladder` | Core P01-B methodology                                    |

---

## 10-Paper Research Programme

This is a PhD/postdoc-scale programme running ~48 months. Before scaffolding new code, check which stage it belongs to. **Do not build Stage 2/3 infrastructure until Stage 0 is complete.**

### Current priority (Stage 0)

Strategic reorganisation of the trajectory programme into two companion papers:

- **P01-A (JRSS-A):** applied paper combining VR-PH regimes with Mapper
  interior structure
- **P01-B (JRSS-B):** methods paper combining the Markov memory ladder with
  survey-design diagnostics

Blocking Phase 0 deliverables:

- authorship / corresponding-author decision
- shared notation standard before draft assembly
- Wasserstein-order audit: the legacy P01 manuscript currently writes `W_1`,
  while reusable trajectory Wasserstein code paths default to `W_2`

P01-A and P01-B should be submitted to JRSS and posted to arXiv on the same day.

### Stage 0 — P01-A / P01-B companion papers (months 0–3)

- **What:** split the former P01/P02/P03 line into an applied JRSS-A paper and a
  methods JRSS-B paper
- **Key outputs:** `papers/P01-A-JRSSA/`, `papers/P01-B-JRSSB/`, and
  `papers/shared/notation.md`
- **Target:** simultaneous JRSS-A + JRSS-B submission with same-day arXiv posting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stephendor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
