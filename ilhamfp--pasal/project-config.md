---
trigger: always_on
description: Pasal.id — Open, AI-native Indonesian legal platform. MCP server + web app giving Claude grounded access to Indonesian legislation.
---

# CLAUDE.md

Pasal.id — Open, AI-native Indonesian legal platform. MCP server + web app giving Claude grounded access to Indonesian legislation.

**Repo:** `ilhamfp/pasal` | **Live:** https://pasal.id | **MCP:** Deployed on Railway

## Architecture

Monorepo with three main pieces:

| Component | Path | Tech |
|-----------|------|------|
| Web app | `apps/web/` | Next.js 16 (App Router), React 19, TypeScript, Tailwind v4, shadcn/ui |
| MCP server | `apps/mcp-server/` | Python 3.12+, FastMCP, supabase-py |
| Data pipeline | `scripts/` | Python — crawler, parser (PyMuPDF), loader, Gemini verification agent |
| Database | `packages/supabase/migrations/` | Supabase (PostgreSQL), 56 migrations (001–055, two 030s + two 039s) |

### Key directories

```
apps/web/src/app/[locale]/     — Public pages under locale segment (/, /search, /jelajahi, /peraturan/[type]/[slug])
apps/web/src/app/admin/        — Admin pages (NOT under [locale], Indonesian only)
apps/web/src/components/       — React components (PascalCase.tsx)
apps/web/src/lib/              — Utilities, Supabase clients (server.ts, client.ts, service.ts)
apps/web/src/i18n/             — i18n config (routing.ts, request.ts)
apps/web/messages/             — Translation files (id.json, en.json)
apps/mcp-server/server.py      — MCP tools: search_laws, get_pasal, get_law_status, list_laws
server.json                    — MCP server registry manifest (name, tools, URL)
scripts/crawler/               — Mass scraper for peraturan.go.id
scripts/parser/                — PDF parsing pipeline (PyMuPDF-based)
scripts/agent/                 — Gemini verification agent + apply_revision()
scripts/loader/                — DB import scripts
packages/supabase/migrations/  — All SQL migrations (001–053)
```

## Commands

```bash
# Web (from apps/web/)
npm run dev          # Dev server
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Vitest

# MCP server (from apps/mcp-server/)
python server.py     # Start MCP server (needs SUPABASE_URL + SUPABASE_ANON_KEY)

# Scraper worker (from project root)
python -m scripts.worker.run  # Background job processor
```

Migrations are applied directly to Supabase via the SQL editor or `supabase db push` — they are not run locally.

## Database Schema

Core tables — all have RLS enabled with public read policies for legal data:

| Table | Purpose |
|-------|---------|
| `works` | Individual regulations (UU, PP, Perpres, etc.). Has `slug`, metadata, parse quality fields. `search_text` maintained by trigger `trg_works_search_text`, `search_fts` TSVECTOR GENERATED ALWAYS from it |
| `document_nodes` | Hierarchical document structure: BAB > Bagian > Pasal > Ayat. Content in `content_text`, `fts` TSVECTOR column auto-generated for search |
| `revisions` | **Append-only** audit log for content changes. Never UPDATE or DELETE rows |
| `suggestions` | Crowd-sourced corrections. Anyone submits, admin approves |
| `work_relationships` | Cross-references between regulations |
| `regulation_types` | ~26 regulation types (UU, PP, PERPRES, UUD, PERPPU, PERMEN, PERDA, etc.) |
| `crawl_jobs` | Scraper job queue and state tracking |
| `scraper_runs` | Scraper session tracking (jobs discovered/processed/failed) |
| `discovery_progress` | Crawl freshness cache per regulation type |

### Critical invariant: content mutations

**Never UPDATE `document_nodes.content_text` directly.** All mutations go through `apply_revision()` (SQL function in migration 020, updated in 038; Python wrapper in `scripts/agent/apply_revision.py`):

1. INSERT into `revisions` (old + new content, reason, actor)
2. UPDATE `document_nodes.content_text` (the `fts` TSVECTOR column auto-updates via `GENERATED ALWAYS`)
3. UPDATE `suggestions.status` if triggered by a suggestion

All steps run in a single transaction. If any fails, everything rolls back.

### Search: `search_legal_chunks()`

3-layer search (migration 039, perf-optimized in 043). Layer 1: **Identity fast path** — detects regulation identifiers (e.g. "uu 10 2011", "uud 1945") via code/name_id match + number extraction, returns deterministic score 1000. **Early exit** — if identity match found, skips Layers 2-3. Handles codes, two-word codes (TAP_MPR), aliases (PERPU→PERPPU), and full name_id prefixes ("Undang-Undang Nomor 10"). Input sanitized (`[^a-zA-Z0-9 ]` → space) to prevent tsquery crashes. Layer 2: **Works FTS** — searches `works.search_fts` for title/topic queries ("ketenagakerjaan"), score ~1-15. Early exit if enough results. Layer 3: **Content FTS** — 3-tier fallback on `document_nodes.fts` (`websearch_to_tsquery` > `plainto_tsquery` > `ILIKE`), score ~0.01-0.5. Uses **CTE pattern**: candidates (capped at 500) → rank → ts_headline only on top N results (avoids O(N) snippet generation). Tier 3 ILIKE capped at 200 candidates. Results accumulate via `RETURN QUERY`; client `groupChunksByWork()` deduplicates by work_id keeping highest score. The function name is intentionally preserved — 5 consumers call it via `.rpc("search_legal_chunks")`.

## Coding Conventions

### TypeScript / Next.js

- **Server Components by default.** Only `"use client"` for interactivity.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilhamfp/pasal](https://github.com/ilhamfp/pasal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
