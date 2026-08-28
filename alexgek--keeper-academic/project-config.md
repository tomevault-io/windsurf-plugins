---
trigger: always_on
description: Pocket-replacement app for saving/reading web articles and academic papers. Runs on Cloudflare Workers + D1 (serverless SQLite). AI features via local Ollama. Zotero integration for academic reading management.
---

# Keeper - CLAUDE.md

## What is this?
Pocket-replacement app for saving/reading web articles and academic papers. Runs on Cloudflare Workers + D1 (serverless SQLite). AI features via local Ollama. Zotero integration for academic reading management.

## Architecture

### Backend (Cloudflare Workers)
- **Runtime**: Cloudflare Workers with Hono router (`src/index.js`)
- **Database**: Cloudflare D1 (SQLite-compatible), database name `keeper-db`
- **Static assets**: Served from `public/` via Workers Sites (`assets = { directory = "public" }` in wrangler.toml)
- **Content extraction**: `linkedom` + `@mozilla/readability` (NOT jsdom - it doesn't work on Workers)
- **FTS**: Manual FTS5 sync required - D1 doesn't support triggers, so `database.js` manually inserts/deletes FTS rows

### Frontend (vanilla JS, no build step)
- `public/app.js` - Main UI logic, auth, article management
- `public/ollama.js` - Ollama client (browser-side, talks to localhost)
- `public/styles.css` - Dark/light theme via CSS variables, responsive breakpoints at 768px and 480px
- `public/mobile.html` - Standalone mobile save page with embedded CSS/JS
- No framework, no bundler. All files served as-is.
- **DOMPurify** and **pdf.js** are vendored in `public/vendor/` (unmodified npm dist builds), not CDN-loaded — so `script-src` is `'self' 'unsafe-inline'` with no third-party hosts. See `SECURITY.md` for how to refresh a version

### Auth
- Token-based: `KEEPER_TOKEN` Cloudflare secret
- Middleware in `src/index.js` checks `Authorization: Bearer <token>` header only (query param support removed for security)
- Frontend stores token in `localStorage` as `keeper_token`
- **Fails closed**: with no `KEEPER_TOKEN` set, `/api/*` returns `503`, not open access. Local dev opts in explicitly with `ALLOW_UNAUTHENTICATED = "true"` in `wrangler.toml`, or put a token in `.dev.vars`
- Failed attempts are throttled per-IP in isolate memory (10/min → `429`). A speed bump, not a guarantee — isolates are per-colo and ephemeral, so an attacker who reconnects gets a fresh budget; a Cloudflare Rate Limiting rule is the real control

### AI (Ollama - client-side only)
- All LLM calls go from browser directly to `localhost:11434` (Ollama)
- Server has zero AI dependencies - it only stores results in `article_insights` table
- Ollama settings stored in `localStorage` (`keeper_ollama_url`, `keeper_ollama_model`, `keeper_ollama_ctx`, `keeper_ollama_num_predict`)
- **Model fallback**: `resolveModel()` in `ollama.js` checks the configured model against `/api/tags` before generating. A model deleted from Ollama leaves a live `localStorage` key holding a dead name, so it falls back to another tag of the same family, then the default, then anything installed — persisting the correction and notifying via `onModelFallback`.
- Background generation: insight jobs continue even when user navigates away from article; results save to server and toast notification shown
- **Two insight modes**: web articles get standard summary/key_points/tags; Zotero items get academic prompt (methodology, key_findings, limitations, relevance). Selection is automatic based on `article.source`
- **Batch insights**: Sequential queue processes multiple articles one-at-a-time with progress bar and cancel support. Uses AbortController to cleanly abort in-flight Ollama requests.
- **Range Vibe enrichment**: When articles have existing AI summaries in `article_insights`, Range Vibe includes those summaries (truncated to 200 chars) in the prompt for richer pattern analysis
- **num_predict**: Configurable max output tokens in AI Settings modal. Stored in `localStorage`. Lower values = faster generation on constrained hardware. Defaults to `OLLAMA_DEFAULTS.numPredict` (1024) when unset; `0` means unlimited.
- **CORS requirement**: Ollama must be started with `OLLAMA_ORIGINS="*"` to accept requests from the Workers domain

### Zotero Integration
- **Server-side only**: `src/zotero.js` talks to Zotero Web API v3 from Workers. No browser-side Zotero calls.
- **Credentials**: Stored in D1 `settings` table (key-value), NOT in localStorage or env vars. API key + user ID.
- **No full-library sync**: By design. Three manual import modes: date range (by `dateAdded`), collection, search.
- **Preview-then-import flow**: Preview endpoints return lightweight item summaries; import endpoint re-fetches full items by key from Zotero API before saving. This avoids shape mismatches between preview and import data.
- **Early termination on date range**: `fetchItemsByDateRange` paginates items sorted by `dateAdded` desc and stops once past the start date, avoiding full-library downloads for large Zotero libraries.
- **Dedup**: Import checks both `zotero_key` and `url` to avoid duplicating items already in Keeper.
- **Full-text**: `fetchFullText(itemKey)` attempts to pull indexed PDF text from Zotero; falls back to abstract-only display if unavailable.
- **PDF attachments**: `fetchPdfFile(attachmentKey)` downloads PDF binary from Zotero Web API. Only works for cloud-synced PDFs — local-only files return 404.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexgek/keeper-academic](https://github.com/alexgek/keeper-academic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
