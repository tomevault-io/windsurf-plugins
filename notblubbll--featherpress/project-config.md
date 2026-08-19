---
trigger: always_on
description: OpenAI-compatible proxy for [Featherless.ai](https://featherless.ai). Preconfigured with a single API key (`rc_...`) in `.config/config.json` and a hardcoded 3-model catalog. Built to survive Featherless's 32k context ceiling via opencode's native subagents (`explore`, `general`) plus the mind MCP for durable memory, with a FIFO queue enforcing the 1-concurrent-call cap (see "Session Memory Protocol" below).
---

# Featherless-Proxy — Developer Guide

OpenAI-compatible proxy for [Featherless.ai](https://featherless.ai). Preconfigured with a single API key (`rc_...`) in `.config/config.json` and a hardcoded 3-model catalog. Built to survive Featherless's 32k context ceiling via opencode's native subagents (`explore`, `general`) plus the mind MCP for durable memory, with a FIFO queue enforcing the 1-concurrent-call cap (see "Session Memory Protocol" below).

## Project Structure

```
FRESHH/
├── proxy.js              # Main proxy implementation + session endpoints (main lane only)
├── dashboard.html        # Dashboard (cache stats, key pool, test chat, wallpapers)
├── .config/
│   └── config.json       # Runtime configuration (API key, hardcoded models, wallpapers, etc.)
├── .cache/
│   ├── sessions/         # Main session log: main.jsonl (subagents use mind MCP, not the proxy)
│   ├── i18n/             # Cached UI translations per locale
│   ├── wallpaper*.jpg    # Cached wallpapers (Bing / Wallhaven / FreeGen)
│   └── usage.db          # (legacy, currently unused — Featherless has no app-usage API)
├── .logs/                # HTTP error logs
├── package.json
├── start.cmd             # Auto-detect launcher (Bun preferred, Node fallback)
├── start-node.cmd        # Node-only launcher
├── skills.md             # Opencode provider configuration reference
├── README.md             # User documentation
└── AGENTS.md             # This file
```

## Constants & Config

- `FEATHERLESS_API_BASE` — `https://api.featherless.ai/v1`
- `API_KEY_ENV_VAR` — `FEATHERLESS_API_KEY`
- `HARDCODED_MODELS` — 3 entries: `moonshotai/Kimi-K3`, `deepseek-ai/DeepSeek-V4-Flash-0731`, `zai-org/GLM-5.2`
- `I18N_TRANSLATE_MODEL` — `zai-org/GLM-5.2` (used for dashboard autotranslation)
- `loadConfig()` — Loads `.config/config.json` with env var overrides
- `saveConfig()` / `debouncedSaveConfig()` — Writes config (debounced 500ms)

## Key Components

### 1. Hardcoded Model Catalog

The model list is hardcoded in `proxy.js` (`HARDCODED_MODELS`). Each entry has `id`, `name`, `reasoning`, `context`, `output`, `tool_call`, `vision`. The proxy does NOT call Featherless's `/v1/models` to populate the catalog at boot — it only calls it to validate the API key (`validateApiKey()`).

`/v1/models` returns the hardcoded list in OpenAI format. `setupOpencodeConfig()` writes all 3 `HARDCODED_MODELS` entries into every discovered `opencode.json`. Subagents use opencode's native `explore`/`general` types (no special model ids) and write findings to mind MCP directly.

### 2. Shell-Tool Guard

`isGitCommand(cmd)` detects shell commands that invoke `git` and rewrites matched tool_calls to `echo "BLOCKED: git commands are disabled by proxy policy"`. Applies to non-streaming responses, streaming SSE responses (buffered and re-emitted), and cached responses on cache hit.

### 3. Retry Logic

`retryLoop(fn)` retries the upstream `/v1/chat/completions` request up to `MAX_RETRIES` (10) with escalating delays (3s, 6s, 9s, …). Retries on HTTP 500, 503, and network failures. On each retry the current key is marked unhealthy so the key pool rotates. Non-retryable HTTP errors are returned immediately.

### 4. Response Cache

LRU cache for non-streaming LLM responses. Key: MD5 of `(model + stream_flag + system + messages + tools)`. TTL default 60s, max size 100. Stats via `GET /api/cache`; clear via `DELETE /api/cache`. Cached responses are passed through the shell-tool guard before being returned.

### 5. Key Pool

Round-robin multi-key pool with cooldown/unhealthy marking. Used when `config.keys` has multiple entries; otherwise the single `config.apiKey` is used. `acquire()` returns `{ key, name, index }` and sets `config.apiKey` + `upstream.apiKey`. `markUnhealthy(index, status)` cools down 60s (503), 30s (502), or 10s (other).

### 6. Upstream Client

`UpstreamClient` with `getModels()` (GET `/v1/models`, 10s timeout) and `chatCompletions(body)` (POST `/v1/chat/completions`). Uses `UPSTREAM_AGENT` — keep-alive HTTPS agent (128 sockets, 60s keepalive). Sends `HTTP-Referer: https://featherless.ai/` and `X-Title: Featherless Proxy` headers as Featherless requests for client attribution.

### 7. Tool Schema Normalization

Normalizes JSON Schema in tools to handle `$ref`, `$defs`, `definitions`, nullable patterns. Applied to `payload.tools` when any tool has `$ref`/`$defs`/`$definitions` in its parameters.

### 8. Stream/Body Utilities

`readBodyText(body)` handles Node streams, Web ReadableStreams, and async iterables. `pipeBodyToResponse(body, res)` pipes with abort handling. SSE streams are buffered through `sanitizeSseResponse()` so the shell-tool guard can intercept tool_call deltas.

### 9. Sleev Context-Compression Gateway (optional)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notBlubbll/featherpress](https://github.com/notBlubbll/featherpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
