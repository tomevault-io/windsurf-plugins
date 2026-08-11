---
trigger: always_on
description: PiPiMink is a Go HTTP service that routes each incoming prompt to the LLM most likely to produce the **best output** for that specific request. The primary design goal is **output quality, not token cost**.
---

# PiPiMink — Project Context for AI Coding Agents

## What this project is

PiPiMink is a Go HTTP service that routes each incoming prompt to the LLM most likely to produce the **best output** for that specific request. The primary design goal is **output quality, not token cost**.

It supports any OpenAI-compatible API (OpenAI, Gemini, OpenRouter, local models via Ollama/llama.cpp/MLX) and Anthropic Claude natively. Azure AI Foundry is supported via multiple single-model provider entries. Exposes drop-in compatible APIs for OpenAI and Ollama clients.

Prefer surgical, test-backed changes over broad refactors.

## How routing works (the core idea)

There are two distinct LLM calls per routed request:

### Step 1: Capability tagging (during model refresh)

Each model is asked to self-assess its own strengths and weaknesses. The model replies with a JSON tag list that is persisted in PostgreSQL:

```json
{ "strengths": ["code-generation", "step-by-step-reasoning"], "weaknesses": ["real-time-information"] }
```

This is triggered by `POST /models/update` (requires `X-API-Key` header) or `./scripts/update_models.sh`.

### Step 2: Prompt-based routing (per request)

When a chat request arrives, a configurable meta-model (`MODEL_SELECTION_MODEL`, default `gpt-4-turbo`) receives the user's prompt plus all enabled models' capability tags. It returns a structured routing decision:

```json
{
  "modelname": "gpt-4o",
  "reason": "...",
  "matching_tags": ["code-generation"],
  "tag_relevance": { "code-generation": 9 }
}
```

The original prompt is then forwarded to the selected model and its response is returned to the caller.

Routing decisions are cached in memory (LRU + TTL) to avoid redundant meta-model calls for similar prompts.

## Key source files

| File | Purpose |
| --- | --- |
| `cmd/server/handlers.go` | HTTP handlers — `handleChat`, `handleOpenAIChatCompletions`, `handleUpdateModels` |
| `cmd/server/server.go` | Server struct, route setup, startup |
| `cmd/server/models.go` | `fetchAndTagModels` — orchestrates model refresh |
| `cmd/server/status_handler.go` | `GET /admin/status` — unauthenticated instance state for setup wizard |
| `cmd/server/console.go` | React SPA serving (embedded via `web/embed.go`) |
| `cmd/server/oauth_handlers.go` | OAuth2/OIDC login, callback, session management, OIDC discovery with retry |
| `cmd/server/auth_middleware.go` | Centralized auth middleware — 3-tier auth (Public/User/Admin), Bearer token validation |
| `cmd/server/auth_admin_handlers.go` | User/group/audit admin API handlers |
| `cmd/server/token_handlers.go` | Per-user Bearer token CRUD (`POST/GET/DELETE /auth/tokens`) |
| `cmd/server/analytics_handlers.go` | Analytics summary and routing decision log |
| `cmd/server/settings_handlers.go` | Settings GET/PATCH handlers |
| `cmd/server/apikeys_handlers.go` | API key vault management |
| `cmd/server/providers.go` | Provider CRUD handlers |
| `cmd/server/config_handlers.go` | Benchmark task and system prompt config |
| `cmd/server/ollama_handlers.go` | Ollama-compatible endpoint handlers |
| `internal/llm/model_selection.go` | `DecideModelBasedOnCapabilities` — the meta-routing call |
| `internal/llm/model_tags.go` | `GetModelTags` — per-model self-assessment call |
| `internal/llm/chat.go` | `ChatWithModel` — forwards prompt to the selected model |
| `internal/llm/decision_cache.go` | In-memory LRU+TTL routing decision cache |
| `internal/llm/client.go` | `Client` struct, provider map, helpers |
| `internal/llm/model_list.go` | `GetModelsByProvider` — list models per provider |
| `internal/config/config.go` | `Config` + `ProviderConfig`; loads `providers.json` + env |
| `internal/config/dotenv.go` | `.env` file read/write helpers |
| `internal/config/settings_registry.go` | Settings registry for runtime config |
| `providers.example.json` | Template for provider configuration |
| `internal/database/database.go` | PostgreSQL persistence (model metadata, tags, auth tables, benchmark results, routing decisions) |
| `internal/models/models.go` | Domain types: `ModelInfo`, `ChatRequest`, `ChatResponse` |
| `internal/benchmark/` | Benchmark task definitions, runner, scorer, suite |
| `web/console/` | React frontend source (TypeScript, Tailwind CSS, React Router) |
| `docs/` | Generated Swagger/OpenAPI artifacts |

Note: `internal/api/` contains only request validators; the server implementation lives in `cmd/server/`.

## API surface

| Endpoint | Description |
| --- | --- |
| `POST /chat` | Native PiPiMink chat — always routes automatically |
| `GET /models` | List all models and their metadata |
| `POST /models/update` | Trigger model refresh (admin, requires `X-API-Key`) |
| `POST /models/discover` | Discover models from all configured providers |
| `POST /models/tag` | Tag selected models (background operation) |
| `GET /models/tag/status` | Tagging progress polling |
| `POST /models/benchmark` | Run benchmarks on selected models (background) |
| `GET /models/benchmark/status` | Benchmark progress polling |
| `PATCH /models/{name}/enable` | Toggle model enabled/disabled |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Izzetee/PiPiMink](https://github.com/Izzetee/PiPiMink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
