---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.


## Commands

```bash
npm install       # install dependencies
npm run dev       # start local dev server (http://localhost:3000)
npm run build     # production build
npm run lint      # run ESLint
npm run test      # run Vitest once
npm run test:watch # run Vitest in watch mode
```

## daisyUI reference
Use this for all UI components: @docs/llms.txt

## Architecture

**Stack:** Next.js 16 (Pages Router) + TypeScript, deployed serverless on Vercel. Tests via Vitest.

**Flow:** cron-job.org (1 min) → `/api/sync` → Upwork API → Zod validation → Supabase log → Notion upsert → Pino logs

### Key files

| Path | Purpose |
|------|---------|
| `src/pages/api/sync.ts` | Main sync entry point (GET only; triggered every 1 min by cron-job.org) |
| `src/pages/api/upwork/auth.ts` | Starts Upwork OAuth2 flow — redirects user to Upwork |
| `src/pages/api/upwork/callback.ts` | Receives OAuth code, exchanges for tokens with retry logic, saves to Supabase |
| `src/pages/api/upwork/fetch.ts` | Calls Upwork REST API via `callUpwork` helper |
| `src/pages/api/upwork/gql.ts` | Proxy to Upwork GraphQL endpoint (`https://api.upwork.com/graphql`) |
| `src/lib/upworkToken.ts` | OAuth token lifecycle: load from Supabase, auto-refresh when <2 min left |
| `src/lib/upworkClient.ts` | `callUpwork()` — authenticated REST wrapper using `getValidAccessToken` |
| `src/lib/notion.ts` | `upsertToNotion()` — find-or-create Notion pages keyed on `External ID` |
| `src/lib/supabase.ts` | `getSupabase()` — lazy-init Supabase client (service role, no session persistence) |
| `src/lib/upwork.ts` | Zod schema for `UpworkItem`; `fetchUpworkItems()` via Upwork GraphQL API |
| `src/lib/requireAuth.ts` | `requireAuth()` — API route guard checking `Authorization: Bearer <API_SECRET>` |
| `src/lib/logger.ts` | Pino logger; pretty-prints in dev, JSON in prod |

### OAuth token storage

Upwork OAuth tokens are stored **per user** in the Supabase `upwork_tokens` table, keyed by `user_id`. `getValidAccessToken()` transparently refreshes when expiry is within 2 minutes.

### Notion upsert

`upsertToNotion()` queries the Notion database by the `External ID` rich-text property to detect existing pages, then either updates or creates. Uses `notion.request()` directly for the query to stay compatible with SDK v5.

### Notion databases

The app uses **3 separate Notion databases**. Each must be shared with the Notion integration (open the DB → top-right `...` → **Connections** → add your integration).

Get a database ID from its URL: `https://notion.so/workspace/`**`<32-char-hex>`**`?v=...`

#### 1. Filtered Job Feed (`NOTION_JOB_FEED_DATABASE_ID`)

Jobs fetched from Upwork marketplace matching your filters. App writes here.

| Property | Type | Notes |
|----------|------|-------|
| `Name` | Title | Job title |
| `Description` | Rich text | Truncated to 2000 chars |
| `External ID` | Rich text | `job-<upwork_id>` — dedup key |
| `Client` | Rich text | Client's country |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexey72882/UpworkToNotion](https://github.com/alexey72882/UpworkToNotion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
