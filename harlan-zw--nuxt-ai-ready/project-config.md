---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**nuxt-ai-ready** is a Nuxt module that makes websites discoverable by AI agents and LLMs through standardized APIs and protocols.

Key features:
- **llms.txt generation**: Auto-generate `llms.txt` and `llms-full.txt` at build time
- **On-demand markdown**: Any route available as `.md` (e.g., `/about` → `/about.md`)
- **MCP server**: `list_pages` and `search_pages` tools for AI agents
- **Content signals**: Configure AI training/search permissions via Nuxt Robots

## Development Commands

```bash
# Build & Development
pnpm build                    # Build module (stub → prepare → build)
pnpm dev                      # Start playground dev server
pnpm dev:prepare              # Build module + prepare playground

# Testing
pnpm test                     # Run all tests (unit + e2e) - runs prepare:fixtures first
pnpm test:unit                # Run unit tests only (no fixture prep)
pnpm test:e2e                 # Run e2e tests only (includes prepare:fixtures)

# Run single test file (unit tests also in src/**/*.test.ts)
pnpm vitest run test/unit/example.test.ts --project=unit
pnpm vitest run test/e2e/basic.test.ts --project=e2e

# Code Quality
pnpm lint                     # ESLint with auto-fix
pnpm typecheck                # TypeScript type checking (no emit)
```

## Architecture

### Build-time Flow (`src/prerender.ts`)

During prerender, the module:
1. Intercepts HTML output via middleware, converts to markdown using **mdream**
2. Writes page data to `.data/ai-ready/page-data.jsonl` (JSONL format)
3. On `prerender:done`, generates:
   - `llms.txt`: Site summary with LLM resource links
   - `llms-full.txt`: Full markdown content of all pages

### Runtime

- **Middleware** (`src/runtime/server/middleware/`): HTML→markdown conversion for `.md` requests
- **Routes**: `/llms.txt`, `/llms-full.txt` handlers (replaced with static files after prerender)
- **MCP** (`src/runtime/server/mcp/`): Tools and resources for AI agent integration
  - `tools/list-pages.ts`: List all pages with metadata
  - `tools/search-pages.ts`: FTS5 full-text search
  - `resources/pages.ts`: Pages resource

### Database Layer (`src/runtime/server/db/`)

SQLite database via db0 for page storage and FTS5 search (tables prefixed `ai_ready_`):
- **schema.ts**: Table definitions (`ai_ready_pages`, `ai_ready_pages_fts`) with FTS5 triggers
- **index.ts**: Database singleton (`useDatabase()`)
- **queries.ts**: Query functions (`getAllPages`, `searchPages`, `upsertPage`, etc.)
- **dump.ts**: Compressed dump export/import for serverless cold starts

### Runtime Plugins (`src/runtime/server/plugins/`)

- **db-restore.ts**: Restores prerendered data from compressed dump on cold start
- **sitemap-seeder.ts**: Seeds routes from sitemap into DB on first request (with TTL)

### Runtime Indexing Flow

Indexing uses explicit polling triggers (no waitUntil piggybacking):

```
sitemap-seeder → seeds routes on first request (once per TTL)
poll endpoint  → indexes pages on-demand via external cron/CI
scheduled task → auto-indexes via Nitro cron (Cloudflare/native)
```

This ensures only public pages (those in sitemap) are indexed, avoiding auth-gated content.

### Indexing Control Endpoints (when `runtimeSync: true`)

- `GET /__ai-ready/status` - Returns `{ total, indexed, pending, indexNow? }`
- `POST /__ai-ready/restore` - Force restore from prerendered dump:
  - `?clear=false` - Don't clear existing pages first (default: true)
  - Requires `Authorization: Bearer <token>` header if `runtimeSyncSecret` configured
  - Returns: `{ restored, cleared }`
- `POST /__ai-ready/poll` - Process pending pages:
  - `?limit=N` - Max pages per batch (default: 10, max: 50)
  - `?all=true` - Process until complete
  - `?timeout=30000` - Max ms for `all` mode (default: 30s)
  - Requires `Authorization: Bearer <token>` header if `runtimeSyncSecret` configured
  - Returns: `{ indexed, remaining, errors, duration, complete }`
- `POST /__ai-ready/prune` - Remove stale routes:
  - `?dry=true` - Preview without deleting
  - `?ttl=N` - Override pruneTtl config
  - Requires `Authorization: Bearer <token>` header unless dry run

### IndexNow Endpoints (when `indexNow` configured)

- `GET /{key}.txt` - Key verification endpoint
- `POST /__ai-ready/indexnow` - Manual sync trigger:
  - `?limit=N` - Max URLs to submit (default: 100)
  - Requires `Authorization: Bearer <token>` header if `runtimeSyncSecret` configured
  - Returns: `{ success, submitted, remaining, error? }`

### Scheduled Task (`src/runtime/server/tasks/ai-ready-cron.ts`)

Cron task runs every minute when enabled. `cron: true` auto-enables `runtimeSync`.

```ts
aiReady: {
  cron: true,          // every minute, auto-enables runtimeSync
  indexNow: true,   // optional IndexNow sync
}
```

**Platform support:**
- **Cloudflare/Native**: Uses Nitro's `scheduledTasks` API
- **Vercel**: Auto-configures `vercel.json` crons to call `GET /__ai-ready/cron`
- **Other**: Use external cron to call `GET /__ai-ready/cron`

### Utils
- **utils/indexPage.ts**: Manual indexing utilities (`indexPage`, `indexPageByRoute`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harlan-zw/nuxt-ai-ready](https://github.com/harlan-zw/nuxt-ai-ready) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
