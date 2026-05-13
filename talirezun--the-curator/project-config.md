---
trigger: always_on
description: This file exists so any new Claude session can immediately understand the project state, architecture, known issues, and active design decisions without re-reading git history or debugging from scratch.
---

# The Curator — Development Guide

This file exists so any new Claude session can immediately understand the project state, architecture, known issues, and active design decisions without re-reading git history or debugging from scratch.

---

## What This Project Is

The Curator is a local Node.js web application that ingests text sources (PDF, MD, TXT) and automatically builds an interconnected knowledge wiki. The wiki is stored as plain markdown files, readable by Obsidian as a visual knowledge graph.

**Core loop:**
1. User drops in a source → LLM reads it → writes wiki pages (entities, concepts, summary)
2. Each subsequent ingest updates existing pages instead of duplicating them
3. Obsidian reads the same files → renders a graph where nodes are entities/concepts, edges are `[[wikilinks]]`

**Philosophy:** Compiled knowledge (persistent wiki), not retrieval (RAG). The wiki compounds with every ingest.

---

## Directory Structure

```
src/
  brain/
    ingest.js     — main ingest pipeline (single-pass + multi-phase for large docs)
    files.js      — all filesystem logic: writePage (returns change records v2.5.0+), mergeWikiPage, syncSummaryEntities, injectSummaryBacklinks
    compile.js    — conversation compilation (v2.5.0): turns a chat thread into wiki pages via the same writePage pipeline
    llm.js        — LLM abstraction (Gemini or Claude, auto-detected via config.js)
    chat.js       — multi-turn chat against the wiki
    sync.js       — GitHub sync (git --git-dir / --work-tree)
    health.js     — wiki health scanner + auto-fix (broken links, orphans, folder-prefix, cross-folder dedup, hyphen variants, missing backlinks)
    config.js     — persistent config (.curator-config.json): getApiKeys, setApiKeys, getEffectiveKey, getDomainsDir
  routes/
    ingest.js     — POST /api/ingest (SSE streaming)
    compile.js    — POST /api/compile/conversation (SSE streaming, v2.5.0)
    domains.js    — domain CRUD
    chat.js       — chat endpoints
    wiki.js       — GET /api/wiki/:domain
    health.js     — GET /api/health/:domain, POST /api/health/:domain/fix[-all]
    sync.js       — sync endpoints
    config.js     — Settings/config endpoints (API keys, updates, domains path)
    mcp.js        — My Curator MCP wizard endpoints (config, claude-config, self-test, reveal-config)
  public/         — vanilla JS frontend (no build step; Settings tab hosts the MCP wizard, Health tab, onboarding wizard)
mcp/              — My Curator: local read-only MCP server that bridges the wiki to Claude Desktop
  server.js       — stdio-transport entry point (spawned by Claude Desktop as a child process)
  graph.js        — wiki parser: frontmatter, [[wikilinks]], backlinks, tag inventory (cached in-process)
  storage/
    local.js      — filesystem adapter; resolves domains path from arg/env/.curator-config.json/default
  util.js         — shared helpers: isValidDomain, isValidSlug, normaliseSlug, resolveNodeSlug
  tools/
    index.js      — tool registration hub + response-size guard (900 KB cap with progressive trim)
    domains.js, index-tool.js, search.js, nodes.js, connected.js,
    summary.js, cross.js, overview.js, tags.js, backlinks.js  — 10 tool modules
scripts/
  inject-summary-backlinks.js   — retroactive backlink repair for existing summaries
  fix-wiki-duplicates.js        — one-time entity/concept deduplication
  fix-wiki-structure.js         — one-time migration from non-canonical folders
  bulk-reingest.js              — re-ingest all raw files in a domain
  repair-wiki.js                — comprehensive wiki repair (cross-folder dedup, link normalization, backlinks)
  build-app.sh                  — rebuild The Curator.app from the AppleScript template
domains/
  <domain>/
    CLAUDE.md         — domain schema (system prompt for LLM)
    raw/              — uploaded source files (gitignored, local only)
    wiki/
      entities/       — people, tools, companies, frameworks
      concepts/       — ideas, techniques, principles
      summaries/      — one page per ingested source
      index.md        — master page catalog
      log.md          — chronological ingest history
    conversations/    — saved chat threads (gitignored)
docs/               — user-facing documentation
```

---

## Key Functions (files.js)

| Function | Purpose |
|---|---|
| `writePage(domain, relativePath, content)` | Normalise path → dedup passes A+B → cross-folder dedup (3b) → inject frontmatter → capture pre-write state → merge with existing → strip blanks → dedup bullets → strip folder-prefix links → normalize variant links (5c: entities + concepts + summaries, prefix-tolerant) → write → call injectSummaryBacklinks if summary → return `{canonPath, status, bytesBefore, bytesAfter, sectionsChanged, bulletsAdded}` (v2.5.0+; null on invalid input) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talirezun/the-curator](https://github.com/talirezun/the-curator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
