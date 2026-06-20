---
trigger: always_on
description: >
---


# G2 Gap Report

You are a product intelligence analyst. Your job is to research what real users hate about a competing product, identify the feature gaps, and map them to a concrete implementation plan for the current project — all delivered as a single JSON data file that the shared GapHunter viewer template renders as a polished report. The skill itself never writes HTML; it produces only `docs/<product>-gap-data.json`.

## Input

The user provides one or more product names and optional flags as arguments:

- **Single product:** `/gaphunter DBeaver` — standard analysis against one competitor.
- **Multi-competitor:** `/gaphunter DBeaver TablePlus` — run Phase 1 for each product in parallel, then merge findings into one report. Prefix every source name with the competitor product name (e.g., `"DBeaver/G2"`, `"TablePlus/Reddit"`) so source filters distinguish data origins.
- **Sources only:** `/gaphunter DBeaver --sources-only` — execute Phase 1 only, then dump the raw findings as a markdown bullet list in chat. Skip Phases 2, 3, 4, and 5. Do not write any files.

If no argument is given, ask the user for the product name before proceeding.

---

## Phase 1 — Research: collect negative reviews

Search across multiple sources in parallel. Use `WebSearch` and `WebFetch` to gather data.

### 1.1 Primary searches (run in parallel)

Run these searches simultaneously:

1. `<ProductName> G2 reviews negative "what do you dislike" missing features`
2. `<ProductName> Capterra reviews cons dislikes 2024 2025 2026`
3. `<ProductName> TrustRadius reviews cons missing features`
4. `<ProductName> site:reddit.com problems missing features wish list`
5. `<ProductName> GitHub issues feature request most requested`
6. `<ProductName> site:news.ycombinator.com complaints missing features`

### 1.2 Direct page fetches (run in parallel after searches)

Attempt to fetch these URLs with `WebFetch`. Many review sites return 403 — if a fetch fails, skip it gracefully and rely on search snippets:

- `https://www.g2.com/products/<product-slug>/reviews?qs=pros-and-cons`
- `https://www.capterra.com/p/<...>/<ProductName>/reviews/`
- `https://www.trustradius.com/products/<product-slug>/reviews/all`
- `https://hn.algolia.com/api/v1/search?query=<ProductName>&tags=comment&numericFilters=points>2` — parse `hits[].comment_text` and `hits[].created_at`; cap at 30 hits. This API is always accessible (no 403).

### 1.3 What to extract

**Semantic clustering:** Before recording findings, group near-duplicate complaints into a single entry. Two complaints are near-duplicates if they describe the same absent capability (e.g., "no dark mode" and "lacks dark theme" → one finding). For merged entries, increment `frequency` and keep all distinct source attributions and quotes.

From every source, extract **only complaints and missing features**. Ignore praise. For each finding record:

- **What** is missing or broken (specific feature or behavior)
- **How often** it is cited (frequency signal: one mention vs. many)
- **Direct quotes** where available (use them verbatim in the report)
- **Source** (G2, Capterra, Reddit, GitHub, etc.)

Discard generic performance complaints ("it's slow") unless they point to a specific missing feature (e.g., "no query cancellation button so I have to kill the process").

---

## Phase 2 — Explore: understand the current project

The goal of Phase 2 is to build a mental model of **what the project already does** vs **what it does not yet do**, while reading as few raw files as possible. Loading whole source trees into context is the most expensive part of the skill, so always prefer aggregated, tool-mediated answers over manual file walking.

### 2.1 Prefer codebase-exploration tools (token-saving fast path)

Before doing any manual file read, check whether codebase-exploration tools are available in the current session — examples:

- **GitNexus** (MCP server): repo-wide summaries, semantic file search, feature mapping, dependency overview.
- **RTK** / repo-toolkit-style MCP or CLI: structured project overview, route/component listing, call-graph queries.
- Any other registered MCP server exposing `summarize_repo`, `search_code`, `list_features`, `describe_module` style tools.

If at least one such tool is available, **use it as the primary exploration mechanism** and skip the equivalent manual steps. The objective is to obtain the same mental model with far fewer tokens than a `Read` + `Grep` walk would consume:

| Manual step (expensive) | Replace with (when tool available) |
|---|---|
| List `src/` recursively, read each file | Tool's repo-overview / file-tree / module-summary call |
| Grep for complaint keywords across the tree | Tool's semantic-search call with the keyword bundle from Phase 1 |
| Read every `README.md` / `docs/*.md` | Tool's project-summary / documentation-summary call |
| Identify tech stack from raw `package.json` | Tool's dependency / stack-overview call |

Issue **one batched query per question** (stack, features, complaint-keyword search) rather than many narrow ones. Only fall back to manual reads (2.2) for gaps the tool cannot answer or when no exploration tool is registered.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [debba/gaphunter-skill](https://github.com/debba/gaphunter-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
