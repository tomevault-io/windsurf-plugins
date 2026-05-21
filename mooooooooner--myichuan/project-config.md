---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

A reverse-engineered proxy that wraps `https://beta.magai.co`'s internal chat link as OpenAI- and Anthropic-compatible HTTP APIs, with a multi-account rotation pool and a small admin web portal. Used in CTF/research scenarios.

The two apps in the pnpm workspace:

- `apps/server` — Node/Express TypeScript proxy. The whole proxy is one file: `apps/server/src/index.ts` (~660 lines).
- `apps/web-portal` — React 19 + Vite + Tailwind admin portal. The UI is one component: `apps/web-portal/src/App.tsx`.

Persistent state lives at `apps/server/accounts.json` (overridable via `MAGAI_ACCOUNTS_FILE`). Server config comes from `apps/server/.env` (template in `.env.example`). Both files are sensitive — do not commit them.

## Common commands

Run from the repository root.

```bash
# Start server in watch mode (tsx). Default port 8787.
pnpm --filter @apps/server dev

# Start the web portal (Vite). Default port 5174, proxies /v1 and /health to 8787.
pnpm --filter @apps/web-portal dev

# Run both in parallel
pnpm dev

# Build everything. Server builds to apps/server/dist/server.cjs (esbuild --format=cjs, single file).
pnpm -r build

# Run the built server
node apps/server/dist/server.cjs
```

There is no test suite, no linter, and no formatter configured. The server has no separate type-check script — `tsx` runs TS directly; `pnpm --filter @apps/server build` is what surfaces type errors via the esbuild bundle (esbuild does not type-check, so use `tsc --noEmit -p apps/server` if you need strict type checking).

Smoke-test the running server:

```bash
curl http://127.0.0.1:8787/health
curl http://127.0.0.1:8787/v1/models -H "Authorization: Bearer $PROXY_API_KEY"
curl http://127.0.0.1:8787/v1/chat/completions -H "Authorization: Bearer $PROXY_API_KEY" -H "Content-Type: application/json" \
  -d '{"model":"claude-sonnet-4-6","messages":[{"role":"user","content":"ping"}]}'
```

## Architecture

### Upstream call chain (the core insight)

Every request to `/v1/chat/completions` or `/anthropic/v1/messages` walks this chain inside `index.ts`:

1. **Pick an account** — `chooseAccount()` round-robins through enabled entries in the in-memory `accounts` array (or honors an explicit `accountId` from the request). Pointer is `rrPointer`.
2. **Refresh Supabase access token** — `getSupabaseAccessToken()` POSTs to `${SUPABASE_URL}/auth/v1/token?grant_type=refresh_token`. Caches the JWT until ~30s before `exp`. **If Supabase rotates the refresh token in the response, the new token is written back to `accounts.json` via `persistAccounts()`** — without this, restart loses the credential.
3. **Discover identity + chat + models** — `refreshDiscovery()` is throttled to once per 30s per account. It:
   - extracts `userId` from the access-token JWT payload (`sub`),
   - calls the `next-action` server action (`MAGAI_NEXT_ACTION`, default `40cd...`) on `${MAGAI_BASE_URL}/chat`,
   - reads `rest/v1/chat` and `rest/v1/spark` rows for the user to find an existing `chatId`, `team`, `workspace`, and historical model IDs,
   - probes catalog tables (`ai_model`, `models`, etc.) and merges with `MAGAI_MODEL_CATALOG_JSON` static config.
4. **Get short JWT for `/api/chat`** — `getMagaiShortJwt()` calls the `next-action` again with the access token + cookie and parses `1:"<jwt>"` from the `text/x-component` response. Cached until ~30s before `exp`.
5. **(Optional) create a fresh chat row** — if `MAGAI_ALWAYS_NEW_CHAT=1` or the request sets `newChat`, `createFreshChatId()` POSTs a new row to `rest/v1/chat` (cloning team/workspace/persona from a template chat).
6. **Hit `/api/chat`** — `requestMagaiChat()` POSTs NDJSON-streaming JSON. Important: the body must use the model's **`apiName`** (e.g. `anthropic/claude-4.6-sonnet-20260217`), not the display name — sending the display name returns 200 with empty content. The `apiName` is discovered from `spark.chat_json.modelDisplay` or supplied via `MAGAI_DEFAULT_MODEL_API_NAME`.
7. **Adapt the NDJSON stream** — `proxyNdjsonToOpenAI()` reads `text-delta` events line-by-line and re-emits either OpenAI SSE (`chat.completion.chunk` + `[DONE]`) or Anthropic SSE (`message_start` → `content_block_delta` → `message_delta` → `message_stop`). Non-stream mode buffers and returns one JSON.

### Account model

Each `Account` carries its own credentials (`magaiCookie`, `currentRefreshToken`, optional per-account overrides for every `MAGAI_*` and `SUPABASE_*` env var) plus per-account caches (`cachedSupabaseAccessToken`, `cachedMagaiJwt`, `discovery`). Per-env-var defaults are read once at startup into `DEFAULT_*` constants and applied via `makeAccount()`.

`bootstrapAccounts()` loads `accounts.json` if present; otherwise it builds a single `default` account from `MAGAI_COOKIE` + `SUPABASE_REFRESH_TOKEN` env vars. `persistAccounts()` writes back the serializable subset (caches and discovery state are not persisted).

`scrubAccount()` is the only thing the management endpoints expose — it strips cookies, tokens, and JWTs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mooooooooner/myichuan](https://github.com/mooooooooner/myichuan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
