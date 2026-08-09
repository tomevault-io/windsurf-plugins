---
trigger: always_on
description: Ungate is a Cursor extension that lets you use Claude, ChatGPT, and MiniMax subscriptions instead of paying for API tokens. It consists of a VS Code extension, a local HTTP proxy (Fastify), a Svelte WebUI, and a shared types/constants library.
---

# Ungate — architecture and project structure

## Purpose

Ungate is a Cursor extension that lets you use Claude, ChatGPT, and MiniMax subscriptions instead of paying for API tokens. It consists of a VS Code extension, a local HTTP proxy (Fastify), a Svelte WebUI, and a shared types/constants library.

## Monorepo (pnpm workspace)

```
ungate/
├── apps/
│   ├── api/          # Local proxy server (Fastify + TypeScript)
│   ├── extension/    # VS Code / Cursor extension
│   └── web/          # Web UI (dashboard) in Svelte
├── packages/
│   ├── shared/       # Shared types, schemas, constants, helpers
│   └── dev-kit/      # Linter, vitest configs
└── scripts/          # Utilities (build, tunnel, check-tunnels)
```

## Data flow

Cursor → Cloudflare Tunnel → Ungate API → Provider API. Cursor cannot call localhost, so a public tunnel URL is required. Ungate listens on a custom OpenAI Base URL, transforms requests into the target provider format and back.

## apps/api — proxy server

Fastify server, spawned by the extension as a child Node.js process.

**Entry points:**
- `/v1/chat/completions` — OpenAI-compatible endpoint, accepts requests from Cursor
- `/v1/messages` — Anthropic-compatible endpoint (direct proxy)
- `/v1/analytics` — request statistics
- `/v1/auth/*` — OAuth routes (Claude, ChatGPT)
- `/v1/models` — model list
- `/v1/settings` — settings

**Architecture:**
- `src/server.ts` — Fastify initialization, plugin registration
- `src/config.ts` — static configuration (URLs, OAuth, beta parameters)
- `src/plugins/auth.ts` — preHandler for API-key authentication

**Routing (`src/routes/`):**
- `openai.ts` — main router: determines provider by model (MiniMax → mapped → Claude). Branch order matters and must stay consistent.
- `anthropic.ts` — direct proxy to Anthropic API with OAuth token
- `analytics.ts`, `auth.ts`, `health.ts`, `models.ts`, `settings.ts`

**Provider selection (`src/orchestration/openai/model-routing.ts`):**
- MiniMax: model name prefix or `model_mappings` entry pointing to MiniMax upstream
- OpenAI-mapped: `model_mappings` entry pointing to OpenAI (ChatGPT Codex)
- Claude (Anthropic): default fallback path
- Each model ID maps to one provider (configured per-row in `model_mappings`), not a global setting.

**Proxy clients (`src/proxy/`):**
- `anthropic-client.ts` — requests to Anthropic Claude Code API via OAuth. Tool name mapping (Cursor → Claude Code), 401 retry with token refresh, 400 error diagnostics, logging for rare `illegal value` errors
- `openai-client.ts` — requests to ChatGPT Codex API (`/responses`). Streaming format converter for both `function_call` and `custom_tool_call` variants
- `minimax-client.ts` — requests to MiniMax API
- `proxy-client.ts` — wrapper
- `tool-mapper.ts` — bidirectional tool name mapping (Cursor ↔ Claude Code)
- `request-builder.ts` — request body preparation for Claude Code (headers, stainless headers, beta params)

**OpenAI orchestration (`src/orchestration/openai/`):**
- `model-routing.ts` — provider determination by model ID
- `provider-handlers/` — handlers for Claude, OpenAI, MiniMax
- `streaming-gateway.ts` — stream management
- `error-mapper.ts`, `error-messages.ts` — error mapping to OpenAI format

**Adapters (`src/adapter/`):**
- `anthropic-to-openai.ts` — Anthropic response → OpenAI format conversion (including streaming tool calls)
- `openai-to-anthropic.ts` — OpenAI request → Anthropic conversion
- `xml-tool-parser.ts` — XML tool use parsing

**Streaming (`src/streaming/`):**
- `openai-stream-handler.ts` — reverse tool name mapping in streams
- `minimax-stream-handler.ts` — MiniMax streaming with `<think>`/`</think>` tag parsing (split tags across chunks via `pendingTag` state)

**Codex responses (`src/proxy/responses-stream-mapper/`):**
- SSE stream parsing from ChatGPT Codex `/responses` into OpenAI chat.completion.chunk
- Handles both streaming variants: `function_call` + `response.function_call_arguments.*` and `custom_tool_call` + `response.custom_tool_call_input.*`
- `stream-state.ts`, `stream-openai-chunks.ts`, `responses-event-router.ts`, `stream-assistant-text.ts`, `process-responses-chunk.ts`, `stream-diagnostics.ts`

**Codex input normalization (`src/proxy/responses-input-normalizer/`):**
- OpenAI chat completion → Codex `/responses` payload transformation
- `input-shape.ts`, `build-body.ts`, `input-text.ts`, `resolve-model.ts`, `types.ts`

**Database (`src/database/`):**
- SQLite via Drizzle ORM + better-sqlite3
- Tables: `app_settings` (port, apiKey, quiet, extraInstruction), `provider_settings` (per-provider OAuth tokens, refresh tokens, expiry), `model_mappings` (id, label, provider, upstreamModel, reasoningBudget), `requests` (analytics)
- Migrations: `apps/api/drizzle/` — idempotent (`CREATE TABLE IF NOT EXISTS`, `INSERT OR IGNORE`)

**Types (`src/types/`):**
- `openai.ts`, `anthropic.ts`, `anthropic-stream.ts`, `proxy.ts`, `auth.ts`

**Tools (`src/tools/`):**
- `translator.ts` — tool name translation in responses
- `normalizer.ts` — parameter normalization (string→array)

**Metrics (`src/metrics/`):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orchidfiles/ungate](https://github.com/orchidfiles/ungate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
