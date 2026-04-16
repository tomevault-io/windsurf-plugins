---
trigger: always_on
description: A minimal LLM chat app built with Next.js 15, deployed on Vercel. The guiding constraints are: as few files as possible, as few dependencies as possible, no over-engineering.
---

# CLAUDE.md

## What this is

A minimal LLM chat app built with Next.js 15, deployed on Vercel. The guiding constraints are: as few files as possible, as few dependencies as possible, no over-engineering.

## Key commands

```bash
pnpm dev          # local dev server
pnpm build        # production build (run to verify before committing)
pnpm test         # unit tests — run after any change to lib/chat.ts or lib/markdown.ts
pnpm db:migrate   # create DB tables (run once)
```

**Run `pnpm test` as you work** — after any change to `lib/chat.ts`, `lib/markdown.ts`, or `app/api/chat/route.ts`, run the tests to catch regressions.

## Architecture

```
app/
  page.tsx                        # entire chat UI — one client component
  layout.tsx                      # HTML shell, imports CSS + highlight.js theme
  globals.css                     # all styles
  error.tsx                       # error boundary for the main page (client component)
  not-found.tsx                   # custom 404 page
  login/page.tsx                  # Google sign-in page (server component)
  share/[id]/
    page.tsx                      # read-only shared chat view (server component)
    fork-button.tsx               # "continue this chat" button (client component)
    opengraph-image.tsx           # OG image generated from share data
  api/
    auth/[...nextauth]/route.ts   # NextAuth handler
    chat/route.ts                 # streaming LLM proxy (OpenAI, Anthropic, Gemini)
    health/route.ts               # GET — DB connectivity check, for uptime monitors
    shares/route.ts               # POST — create a share
    shares/[id]/route.ts          # GET — fetch a share
    history/route.ts              # GET — list encrypted chat history; POST — save/update
    history/[id]/route.ts         # DELETE — remove a history entry
    settings/route.ts             # GET/PUT — user settings (system prompt, save toggle, key)

lib/
  chat.ts                         # pure message-conversion functions (tested)
  crypto.ts                       # AES-GCM encrypt/decrypt for chat history
  db.ts                           # pg Pool singleton + query helper
  log.ts                          # pino logger instance (import and use directly)
  markdown.ts                     # marked + marked-highlight + highlight.js setup

tests/
  unit.test.ts                    # node:test unit tests for lib/chat.ts + lib/markdown.ts

auth.config.ts                    # lightweight NextAuth config (edge-safe, no providers)
auth.ts                           # full NextAuth config (Google provider) — server only
middleware.ts                     # auth guard using auth.config.ts (keeps middleware bundle small)
scripts/migrate.mjs               # one-shot DB table creation
```

## Dependencies

Runtime: `next`, `react`, `react-dom`, `next-auth`, `marked`, `marked-highlight`, `highlight.js`, `pg`, `pino`

`pg` and `pino` are in `serverExternalPackages` in `next.config.ts` to avoid bundling issues.

## Auth

NextAuth v5 (beta) with Google OAuth. All routes except `/login`, `/share/*`, `/api/auth/*`, and `/api/health` require authentication (enforced in `middleware.ts`). The `ALLOWED_EMAIL` env var is a comma-separated list of permitted emails; anyone else is rejected at the `signIn` callback in `auth.config.ts`.

**Important:** `auth.config.ts` is the lightweight edge-safe config used in middleware — it must include `providers: []` even though it has no providers, or next-auth will crash with a `.map()` error on undefined. `auth.ts` imports `auth.config.ts` and adds the Google provider for server-side use.

## LLM providers

`app/api/chat/route.ts` handles all three providers via their REST APIs directly (no SDKs):

- **OpenAI** — REST SSE, `Authorization: Bearer` header; web search uses the Responses API (`/v1/responses`)
- **Anthropic** — REST SSE, `x-api-key` header, `anthropic-version` header; system prompt is a top-level field; web search uses a multi-turn tool loop (non-streaming rounds, then final text)
- **Google Gemini** — REST SSE via `generativelanguage.googleapis.com`; role `assistant` maps to `model`; system prompt via `systemInstruction`

Images are stored as `{ data: string, mimeType: string }` (raw base64, no data-URL prefix). PDFs are stored as `{ name: string, data: string }` (base64). Both are converted to provider-specific formats in the route.

The chat route validates requests before hitting any upstream API: `messages` must be a non-empty array (max 200), and `model` must be in `ALLOWED_MODELS`.

## Logging

Use `logger` from `lib/log.ts` (pino). Convention:
- `logger.info(fields, 'event.name')` — normal operations
- `logger.warn(fields, 'event.name')` — auth failures, 4xx
- `logger.error(fields, 'event.name')` — 5xx, exceptions

The chat route uses a try/finally to emit one canonical log line per request with: `user`, `model`, `provider`, `msgs`, `webSearch`, `status`, `ms` (time to first stream byte). Set `LOG_LEVEL=debug` in env for more verbose output.

## Chat history


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cartermp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
