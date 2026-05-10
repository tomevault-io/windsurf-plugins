---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Info

- **npm:** https://www.npmjs.com/package/aso-mcp
- **GitHub:** https://github.com/KenanAtmaca/aso-mcp
- **Version:** 1.1.0
- **Install:** `npm install -g aso-mcp` or `npx aso-mcp`

## Build & Run Commands

```bash
npm run dev          # Run server directly with tsx (development)
npm run build        # Compile TypeScript to ./build + chmod 755
npm run start        # Run compiled server (production)
npm run inspect      # Open MCP Inspector in browser for interactive testing
npx tsx test.ts              # Core test suite (17 tests — cache, scoring, scraping, integration)
npx tsx test-generation.ts   # ASO generation scenario tests (8 tests)
npx tsx test-phase3.ts       # Localization & report tests (4 tests)
npx tsx test-connect.ts      # App Store Connect tests (9 tests — locale mapping + optional live API)
```

## Publishing

```bash
npm version patch|minor|major   # Bump version + git commit + tag
npm publish --access public     # Publish to npm (requires auth token)
```

## Architecture

**MCP Server** serving 19 tools over stdio transport. Tools are registered in `src/server.ts` grouped by phase.

**Data flow:** Tool Handler → Zod Validation (min/max constraints) → SQLite Cache check → Rate Limiter (token bucket + exponential backoff retry) → Data Source → Custom Scoring (fallback) → Cache result (max 5000 entries) → Return JSON to client.

### Tools (19)

| # | Tool | Phase | Description |
|---|------|-------|-------------|
| 1 | `search_keywords` | 1 | Keyword traffic/difficulty scores + competitor apps |
| 2 | `suggest_keywords` | 1 | Keyword suggestions (category, similar, competition strategies — parallel execution) |
| 3 | `get_app_details` | 1 | Full app metadata, ratings, reviews count |
| 4 | `analyze_competitors` | 2 | Top-ranking apps metadata comparison + keyword gap |
| 5 | `optimize_metadata` | 2 | Title/subtitle/keyword field optimization suggestions |
| 6 | `analyze_reviews` | 2 | Sentiment analysis (TR+EN with diacritic variants), complaints, feature requests |
| 7 | `track_ranking` | 2 | App ranking position for specific keywords (exact match) |
| 8 | `keyword_gap` | 2 | Keyword difference between two apps (batch scoring) |
| 9 | `localized_keywords` | 3 | Multi-country keyword score comparison (parallel per country) |
| 10 | `get_aso_report` | 3 | Comprehensive ASO report for an app |
| 11 | `discover_keywords` | Gen | Keyword discovery from scratch for new apps |
| 12 | `generate_aso_brief` | Gen | Complete ASO brief with title/subtitle/keyword suggestions |
| 13 | `connect_setup` | 5 | Configure & validate App Store Connect credentials |
| 14 | `connect_get_app` | 5 | Find app by bundle ID, get ASC ID + version state |
| 15 | `connect_get_metadata` | 5 | Read current metadata (name, subtitle, keywords, desc, supportUrl, marketingUrl) for a locale |
| 16 | `connect_update_metadata` | 5 | Write metadata with char limit validation, HTML entity sanitization, cache invalidation + before/after diff |
| 17 | `connect_batch_update_metadata` | 5 | Batch update metadata for multiple locales in one call (max 40 locales) |
| 18 | `connect_list_localizations` | 5 | List all locales and metadata completeness status |
| 19 | `clear_cache` | Util | Clear local SQLite cache |

### Key Layers

- **`src/tools/`** — 19 MCP tool definitions. Each follows the pattern: Zod schema validation (with min/max constraints) → cache lookup → data source calls → format result → cache + return. All return `{ content: [{ type: "text", text: JSON }] }`.
- **`src/data-sources/`** — Four data adapters:
  - `app-store.ts` — Wraps `app-store-scraper` (search, app details, reviews, ratings, suggestions, similar apps). All calls go through rate limiter.
  - `aso-scoring.ts` — Wraps `aso` npm package for traffic/difficulty scores. **Falls back automatically** to custom scoring when the aso package returns 503 (Apple API issue). Fallback has a 10-minute retry timer (`ASO_RETRY_INTERVAL_MS`) — after 10 minutes, the aso package is retried automatically. `batchGetScores()` processes keywords in parallel batches of 5.
  - `custom-scoring.ts` — Four scoring algorithms (visibility, competitive, opportunity, overall) independent of Apple APIs. Also provides `extractTitleKeywords()` with Turkish + English stop word filtering.
  - `app-store-connect.ts` — App Store Connect API client. JWT ES256 auth via `jsonwebtoken` with token caching (~18 min reuse, 2 min safety margin). Manages credentials from `~/.aso-mcp/connect-config.json` or env vars (`ASC_ISSUER_ID`, `ASC_KEY_ID`, `ASC_PRIVATE_KEY_PATH`). Reads/writes metadata via App Info Localizations (name + subtitle) and App Store Version Localizations (keywords, description, promotionalText, whatsNew, supportUrl, marketingUrl). Includes `decodeHtmlEntities()` sanitization and editable appInfo selection logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KenanAtmaca/aso-mcp](https://github.com/KenanAtmaca/aso-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
