---
trigger: always_on
description: Personal semantic thought storage and retrieval MCP server with vector embeddings and ltree scoping. Implements the 8-tool atelier-brain protocol for cross-project compatibility (mybrain ADR-0001).
---

# mybrain-mcp

## Project Overview

Personal semantic thought storage and retrieval MCP server with vector embeddings and ltree scoping. Implements the 8-tool atelier-brain protocol for cross-project compatibility (mybrain ADR-0001).

## Tech Stack

- **Runtime:** Node.js 18+ (ESM modules)
- **MCP Framework:** `@modelcontextprotocol/sdk` 1.27.1
- **Database:** PostgreSQL 12+ with pgvector (1536-dim embeddings) and ltree extensions
- **HTTP Server:** Node `http` (Streamable HTTP transport mode)
- **Embeddings:** Provider-abstracted (OpenRouter, OpenAI, GitHub Models, local Ollama)
- **Containerization:** Docker (Alpine), Docker Compose

## Source Layout

```
server.mjs              — startup orchestrator (transport + wiring only, ~250 lines)
lib/                    — all server logic
  config.mjs            — resolveConfig, buildProviderConfig, identity, enums
  db.mjs                — createPool, runMigrations
  crash-guards.mjs      — installCrashGuards (process-level signal/error handlers)
  embed.mjs             — getEmbedding, probeEmbeddingDim, startEmbedWorker
  llm-provider.mjs      — embed/chat provider adapters (openai-compat, anthropic, local)
  llm-response.mjs      — assertLlmContent (LLM response validation)
  conflict.mjs          — detectConflicts, classifyConflict, getBrainConfig
  consolidation.mjs     — startConsolidationTimer, runConsolidation
  ttl.mjs               — startTTLTimer, runTTLEnforcement
  tools.mjs             — registerTools (8 protocol tools)
  rest-api.mjs          — createRestHandler (Settings UI REST endpoints)
  static.mjs            — handleStaticFile (Settings UI asset serving)
  hydrate.mjs           — discoverSubagentFiles/discoverEvaFiles/Tier 1-3 hydration
migrations/             — auto-applied SQL migrations (idempotent, runs at startup)
scripts/                — out-of-band utilities (hydrate-telemetry.mjs)
templates/              — Docker deployment scaffolding (Dockerfile, compose.yml, schema.sql)
ui/                     — reserved for Settings UI static assets (placeholder; lib/static.mjs serves from here when populated)
tests/brain/            — node:test integration tests (require DATABASE_URL)
skills/                 — Claude Code skills (mybrain-setup, mybrain-overview)
.claude-plugin/         — plugin manifest for marketplace
```

## Run Commands

```bash
node server.mjs           # stdio mode (Claude Code CLI)
node server.mjs http      # HTTP Streamable mode (port 8787)
```

## Database Pattern

Raw SQL with `pg` driver and `pgvector` bindings. No ORM. Embeddings stored as `vector(1536)`. Scope filtering via ltree arrays. SQL is split across `lib/` modules (tools.mjs, conflict.mjs, consolidation.mjs, ttl.mjs, rest-api.mjs, hydrate.mjs).

Schema lives in `templates/schema.sql`. On startup, `runMigrations(pool)` detects an empty database (via `to_regclass('thoughts') IS NULL`) and applies `templates/schema.sql` with `{{EMBED_DIM}}` substituted to `1536`, then runs the numbered migrations. Operators using a non-1536-dim embedding model must apply `schema.sql` by hand with the right substitution before first startup. The auto-bootstrap is recorded as `000-baseline-schema.sql` in `schema_migrations`. Existing v1 mybrain databases skip the bootstrap (the `thoughts` table already exists) and pick up the v1-to-merged migration on the next startup.

## MCP Tools

The server exposes 8 protocol tools (replacing the legacy 4-tool API, see `## Tool Rename (v2.0)` in README.md):

| Tool | Purpose |
|---|---|
| `agent_capture` | Store a thought with schema-enforced metadata; handles dedup, conflict detection, supersedes relations |
| `agent_search` | Semantic search with three-axis scoring (recency + importance + relevance) |
| `atelier_browse` | Paginated thought listing with status/type/agent/scope filters |
| `atelier_stats` | Brain health check + counts by type/status/agent |
| `atelier_relation` | Link two thoughts via a typed relation |
| `atelier_trace` | Traverse the relation graph from a thought |
| `atelier_hydrate` | Hydrate JSONL telemetry from a Claude Code project sessions directory |
| `atelier_hydrate_status` | Poll completion state of a previous `atelier_hydrate` call |

## Environment Variables

```
DATABASE_URL                    — PostgreSQL connection string (or ATELIER_BRAIN_DATABASE_URL)
ATELIER_BRAIN_DATABASE_URL      — alias for DATABASE_URL (atelier-brain compatibility)
OPENROUTER_API_KEY              — OpenRouter API key (legacy single-provider fallback)
ATELIER_BRAIN_API_TOKEN         — REST API auth token (or MYBRAIN_API_TOKEN)
MYBRAIN_API_TOKEN               — alias for ATELIER_BRAIN_API_TOKEN
MYBRAIN_ASYNC_STORAGE           — "true" to enable background embed worker
MYBRAIN_WORKER_POLL_MS          — embed worker poll interval (default: 500)
MYBRAIN_WORKER_BATCH            — embed worker batch size (default: 8)
BRAIN_CONFIG_PROJECT            — explicit path to project brain-config.json
BRAIN_CONFIG_USER               — explicit path to user brain-config.json
ATELIER_BRAIN_USER / MYBRAIN_USER  — override identity for `captured_by` field

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertsfeir/mybrain](https://github.com/robertsfeir/mybrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
