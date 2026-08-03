---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Paperless-AI **next** — a community fork of clusterzx/paperless-ai. A Node.js/Express app that uses
LLMs (OpenAI, Ollama, Azure, or any OpenAI-compatible "custom" endpoint) plus optional OCR to auto-tag,
title, classify, and extract metadata for documents in a Paperless-ngx instance. Server-rendered EJS UI,
no frontend framework. Single SQLite database (better-sqlite3, WAL mode).

## Commands

> ⚠️ **`npm test` / `npm run test` does NOT run tests** — it starts the dev server via nodemon
> (`nodemon server.js`). This is a deliberate quirk of `package.json`.

```bash
# Run the dev server (auto-reload)
npm run test                       # = nodemon server.js
node server.js                     # plain run

# Tests (custom runner — plain node scripts, no jest/mocha)
npm run test:all                   # all tests          (node scripts/run-tests.js --all)
npm run test:list                  # list areas + tests
npm run test:area:security         # run an area (auth|ocr|observability|processing|prompts|security)
node scripts/run-tests.js --test ssrf-url-validation   # single test by name

# Lint / format
npx eslint .                       # flat config: eslint.config.mjs
npx prettier --check .             # .prettierrc.json + .prettierignore

# Production (what Docker runs)
pm2-runtime ecosystem.config.js    # via start-services.sh

# Maintenance
npm run mfa:reset                  # reset MFA for a user
node scripts/regen-openapi.js      # regenerate OPENAPI/openapi.json from @swagger JSDoc
```

**Test gotcha:** several tests self-skip unless a server is reachable at `BASE_URL`
(default `http://localhost:3000`) and certain env vars are set — e.g. `rate-limiting`,
`thumbnail-auth-guard`, `scan-stop-flow` (needs `JWT_TOKEN` or `API_KEY`), `login-mfa-flow`
(needs `LOGIN_TEST_USERNAME`/`LOGIN_TEST_PASSWORD`). A "SKIPPED" result is not a failure. To run
the full server-dependent suite, start the server first, then run the tests against it.

## Architecture

**Entry point:** `server.js` (~1200 lines) — Express app, all middleware/routes/CSRF/rate-limiting,
the cron-driven `scanDocuments()` processing loop, and SSE progress streams. Most logic lives here plus
the `services/` singletons.

**Service layer** (`services/`): every service is a singleton —
`class Foo { ... }; module.exports = new Foo();`. Import and call directly; do not `new` them.

**AI provider abstraction:** `services/aiServiceFactory.js` returns the right service based on
`config.aiProvider` (`openai` | `ollama` | `custom` | `azure`). Every provider service implements
`analyzeDocument(content, doc, existingTags, correspondents)`. To add a provider: add a service singleton,
wire it into the factory switch, and add its config block in `config/config.js`.

**Token handling:** `services/serviceUtils.js` — `calculateTokens()` (tiktoken for OpenAI, ÷4 char
estimate otherwise) and `truncateToTokenLimit()`. Also holds AI-error → OCR-fallback classification
helpers used by the scan loop.

**Prompt restrictions:** `services/restrictionPromptService.js` replaces placeholders like
`%RESTRICTED_TAGS%`, `%RESTRICTED_CORRESPONDENTS%`, `%CUSTOMFIELDS%` in prompts. The base prompts live
in `config/config.js` (`specialPromptPreDefinedTags`, `mustHavePrompt`).

**OCR:** `services/mistralOcrService.js` — provider can be `mistral`, `ollama` (native `/api/chat`
vision), or OpenAI-compatible `/v1`. OCR is a fallback path when AI analysis fails on low-text docs.

**Database:** `models/document.js` — better-sqlite3 at `data/documents.db`, WAL mode, prepared
statements throughout. Key tables: `processed_documents`, `history_documents` (server-side paginated
via SQL `LIMIT/OFFSET`, not in-memory), `openai_metrics`, `original_documents`, `users`.
Schema migrations run at startup via `services/startupMigrations.js`.

**Paperless-ngx I/O:** `services/paperlessService.js` fetches documents and posts results back
(`updateDocument()`). Tags are cached with a TTL (`TAG_CACHE_TTL_SECONDS`, default 300s) to cut API calls.

**Reconciliation:** `services/reconciliationService.js` — cron job (`RECONCILIATION_INTERVAL`) that
cleans up records for documents deleted in Paperless-ngx.

**Routes:** `routes/auth.js` (JWT-in-cookie + `x-api-key` auth, `isAuthenticated` middleware) and
`routes/setup.js` (setup wizard + history/api endpoints). `schemas.js` holds shared swagger schemas.

### Document processing flow (the core loop in server.js)

1. `node-cron` fires on `config.scanInterval` (cron syntax, default `*/30 * * * *`).
2. `scanDocuments()` pulls candidates from Paperless-ngx.
3. A `retryTracker` Map caps attempts (max 3) to prevent infinite loops.
4. Docs below `MIN_CONTENT_LENGTH` (default 10) are skipped or routed to OCR.
5. If `PROCESS_PREDEFINED_DOCUMENTS=yes`, only docs whose tags match the `TAGS` env var are processed.
6. The factory's AI service analyzes; results are written back via `paperlessService.updateDocument()`.
   A global `__paperlessAiScanControl` object supports stop-requests mid-scan.

## Configuration system (important)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [admonstrator/paperless-ai-next](https://github.com/admonstrator/paperless-ai-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
