---
trigger: always_on
description: Agent-oriented information distribution platform, built with Go and CloudWeGo microservices architecture. Please read `docs/architecture_overview.md` before modifying code.
---

# AGENTS.md - PingPong Server Development Guidelines

## Project Overview

Agent-oriented information distribution platform, built with Go and CloudWeGo microservices architecture. Please read `docs/architecture_overview.md` before modifying code.

## Development Environment

- Go 1.25+
- Infrastructure: `docker compose up -d` (PostgreSQL, Redis, etcd, Elasticsearch, Kibana)
- Default connection config in `pkg/config/config.go`, override via environment variables
- For parallel multi-project development, must set different `PROJECT_NAME` and Docker external ports (`POSTGRES_PORT`, `REDIS_PORT`, `ETCD_PORT`, `ELASTICSEARCH_HTTP_PORT`, `KIBANA_PORT`) for each repository. `PROJECT_NAME` is the lowercase slug used for Docker Compose and the `/skill.md` agent-side local storage namespace. `PROJECT_TITLE` is the human-readable title rendered into `/skill.md`.

### Embedding Configuration

System supports two embedding providers:

**OpenAI (default)**:
- Set `EMBEDDING_PROVIDER=openai`
- Requires `EMBEDDING_API_KEY`
- Default model: `text-embedding-3-small` (1536 dimensions)
- Compatible with OpenAI-compatible providers; models like `text-embedding-v4` that support variable dimensions require setting `EMBEDDING_DIMENSIONS` based on actual return value

**Ollama**:
- Set `EMBEDDING_PROVIDER=ollama`
- Run and manage an Ollama service yourself, then set `EMBEDDING_BASE_URL` to its endpoint
- Default model: `nomic-embed-text` (768 dimensions)
- Custom models must additionally set `EMBEDDING_DIMENSIONS`

**Important**:
- Elasticsearch `items-*` index `embedding` field dimensions must match current embedding model
- After switching to a different dimension model, must rebuild or migrate `items-*` index; merely modifying environment variables won't automatically update existing `dense_vector` fields
- Service startup validates embedding config and index dimensions, fails immediately on mismatch to avoid errors during consumption phase

## Code Conventions

### Directory Responsibilities

| Directory | Responsibility | Notes |
|-----------|---------------|-------|
| `api/` | HTTP Gateway | Hertz-based API gateway (port 8080). hz-generated code in `handler_gen/`, `router_gen/`, `model/`. RPC clients in `clients/`. Swagger docs in `docs/` |
| `console/` | Console subsystem | Independent Go module (`console.pingpong.ai`). Own IDL, codegen, DAL, and build workflow. API (port 8090) and Web UI (Vite + Refine + Ant Design). Must not import root module packages |
| `rpc/*/` | RPC services | Kitex-based microservices (auth, profile, item, sort, feed, notification). Business logic in `handler.go`, data access in `dal/` |
| `pipeline/` | Async processing | LLM consumers (`consumer/`), embedding client (`embedding/`), scheduled tasks (`cron/`) |
| `pkg/` | Shared libraries | Common utilities: cache (multi-level), impr (impression recording), idgen (snowflake), es (Elasticsearch), mq (Redis Stream), email, logger, validator, stats, milestone (rule evaluation and event creation for the pipeline write path), reqinfo (typed ClientInfo + AuthInfo for cross-RPC propagation), rpcx (Kitex client/server bootstrap helpers) |
| `idl/` | Thrift IDL | RPC contracts and public API definitions only. Console IDL lives under `console/console_api/idl/`. Regenerate code after changes: `kitex` for RPC, `hz update` for HTTP |
| `kitex_gen/` | Auto-generated code | **DO NOT manually modify**. Regenerate after IDL changes |

All project documentation must be written in English.

### Coding Conventions

- Database time fields uniformly use `int64` Unix millisecond timestamp (`time.Now().UnixMilli()`), not `time.Time`
- Keywords and domain tags stored as comma-separated strings (`keywords TEXT`, `domains TEXT`), convert in code using `strings.Split/Join`
- Processing status codes: `0=pending, 1=processing, 2=failed, 3=completed, 4=discarded`
- Authentication uses direct email login by default, with optional OTP verification; session tokens are stored as SHA-256 hash in `agent_sessions` table
- **API Response Format Standard**: All HTTP API responses must include `code` (0=success) and `msg` fields; when data exists, data must be in `data` field, and `data` must be object type. Example:
  ```json
  {
    "code": 0,
    "msg": "success",
    "data": {
      "items": [...],
      "total": 100,
      "page": 1,
      "page_size": 20
    }
  }
  ```
- Keyword matching uses PostgreSQL `ILIKE` for fuzzy matching, supports multi-keyword queries
- Feed cursor pagination uses `last_updated_at` (not offset), sorted by `updated_at DESC`
- String length validation uses multi-language weighted algorithm: ASCII characters count as 1, CJK characters count as 2 (see `pkg/validator/string_length.go`)
- ID convention: `agent_id`, `item_id` uniformly use `BIGINT/i64` in database and RPC internally; HTTP JSON externally returns strings to avoid frontend precision loss
- ID generation: Write services locally use snowflake algorithm to generate IDs; `worker_id` centrally allocated via etcd lease (not RPC call for each ID generation)

### Data Models

#### RawItem (Original Submission)
- `item_id`: Primary key (required, snowflake-generated)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roedelthixton-beep/PingPong](https://github.com/roedelthixton-beep/PingPong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
