---
trigger: always_on
description: MCP Server cho AI memory. Python 3.13, uv, hatchling, src layout.
---

# AGENTS.md - mnemo-mcp

MCP Server cho AI memory. Python 3.13, uv, hatchling, src layout.
Hybrid search: FTS5 + sqlite-vec semantic. 15 tools: 11 specialized memory tools (add_memory, search_memory, list_memories, update_memory, delete_memory, export_memories, import_memories, memory_stats, restore_memory, archived_memories, consolidate_memories) + legacy `memory` dispatcher (DEPRECATED -- use the granular tools instead) + config + help + config__open_relay.
2-mode embedding: cloud chain (`EMBEDDING_MODELS`) > Local (Qwen3 ONNX khi chain rong). Per-task model chains (`EMBEDDING_MODELS`/`RERANK_MODELS`/`LLM_MODELS`, order = litellm fallback). LLM/Embed/Rerank: litellm passthrough qua `mcp_core.llm` (mcp-core[llm]).

## Commands

```bash
# Setup
uv sync --group dev

# Lint & Type check
uv run ruff check .
uv run ruff format --check .
uv run ty check

# Fix
uv run ruff check --fix .
uv run ruff format .

# Test (integration excluded by default)
uv run pytest
uv run pytest tests/test_db.py -v                          # single file
uv run pytest tests/test_db.py::TestSearch::test_basic -v  # single test

# Build & Run
uv build
uv run mnemo-mcp                    # run server (warmup/setup_sync via config tool)

# Mise shortcuts
mise run setup     # full dev setup
mise run lint      # ruff check + format check + ty check
mise run test      # pytest
mise run fix       # ruff fix + format
```

## Pytest

- `asyncio_mode = "auto"` -- khong can `@pytest.mark.asyncio`
- Timeout: 30s/test
- Integration marker: `@pytest.mark.integration` (can network/services)
- Default: `-m 'not integration and not live and not full'`
- Snapshot testing: syrupy

## Cau truc thu muc

```
src/mnemo_mcp/
  __main__.py      # python -m mnemo_mcp entrypoint
  config.py        # Pydantic Settings (singleton), env vars khong co prefix
  server.py        # FastMCP server, tools, resources, prompts
  setup_tool.py    # Warmup + setup-sync logic (config tool actions)
  db.py            # SQLite: CRUD, FTS5, vector search (sqlite-vec)
  embedder.py      # Dual-backend: multi-provider cloud (Jina/Gemini/OpenAI/Cohere) + qwen3-embed local
  reranker.py      # Dual-backend reranking: cloud (Jina/Cohere) + local (qwen3-embed cross-encoder)
  graph.py         # Knowledge graph: entity/relation extraction via LLM
  relay_setup.py   # Zero-config relay: create session, poll for config
  relay_schema.py  # Relay form schema (local + cloud modes)
  sync/            # Sync backends: gdrive.py (OAuth Device Code, httpx) + s3.py (R2/B2/MinIO) + delta/bundle/base
  token_store.py   # OAuth token storage (secure file-based, chmod 600)
  docs/            # Tool documentation markdown
tests/             # 1:1 mapping voi source modules
```

## Env vars

Khong co prefix (khac voi cac project khac):
- `DB_PATH` -- default `~/.mnemo-mcp/memories.db`
- `EMBEDDING_MODELS` -- chain embedding, CSV `provider/model,provider/model`; order = litellm fallback. Rong = local ONNX (qwen3-embed).
- `RERANK_MODELS` -- chain rerank, CSV `provider/model,...`; order = fallback. Rong = local ONNX cross-encoder.
- `LLM_MODELS` -- chain LLM (graph extraction), CSV `provider/model,...`; order = fallback. Rong = tat feature LLM.
- Provider duoc suy ra tu prefix model. API key theo convention litellm `<PROVIDER>_API_KEY`. 7 provider servers goi y:

  | model prefix | key env var | get it at |
  |---|---|---|
  | `gemini/` | `GEMINI_API_KEY` | aistudio.google.com/apikey |
  | `openai/` (or bare) | `OPENAI_API_KEY` | platform.openai.com |
  | `jina_ai/` | `JINA_AI_API_KEY` | jina.ai/api-key |
  | `cohere/` | `COHERE_API_KEY` | dashboard.cohere.com |
  | `xai/` | `XAI_API_KEY` | console.x.ai |
  | `anthropic/` | `ANTHROPIC_API_KEY` | console.anthropic.com |
  | `vertex_express/` | `GOOGLE_VERTEX_EXPRESS_API_KEY` | cloud.google.com/vertex-ai/generative-ai/docs/start/express-mode/overview |

  For any other litellm provider (used via env passthrough), see https://docs.litellm.ai/docs/providers/<provider> for its `<PROVIDER>_API_KEY` name.
- Custom endpoint (SSRF-guarded): `LLM_API_BASE`, `EMBEDDING_API_BASE`, `RERANK_API_BASE`
- `EMBEDDING_DIMS` -- default 768 (0 = auto)
- Deprecated (honored mot release voi warning): singular `EMBEDDING_MODEL`/`RERANK_MODEL` + `EMBEDDING_BACKEND`/`RERANK_BACKEND` (backend gio suy ra tu chain rong hay khong). Router auto-detect cu "Jina > Gemini > OpenAI > Cohere" da bo.
- `SYNC_ENABLED` -- `true`/`false`, default true
- `GOOGLE_DRIVE_CLIENT_ID` -- OAuth client ID (required for sync)
- `SYNC_FOLDER` -- Google Drive folder name (default: `mnemo-mcp`)
- `SYNC_INTERVAL` -- seconds (0 = manual only, default: 300)
- `RERANK_ENABLED` -- `true`/`false`, default true
- `RERANK_TOP_N` -- so ket qua rerank giu lai (default: 10)
- `ARCHIVE_ENABLED` -- `true`/`false`, default true
- `ARCHIVE_AFTER_DAYS` -- so ngay truoc khi archive (default: 90)
- `ARCHIVE_IMPORTANCE_THRESHOLD` -- nguong importance de giu lai (default: 0.3)
- `DEDUP_THRESHOLD` -- nguong similarity de coi la duplicate (default: 0.9)
- `DEDUP_WARN_THRESHOLD` -- nguong similarity de canh bao (default: 0.7)
- `RECENCY_HALF_LIFE_DAYS` -- half-life cho temporal decay scoring (default: 7)
- `LOG_LEVEL` -- log level (default: INFO)

### Manual config example

```json
{
  "mcpServers": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n24q02m/mnemo-mcp](https://github.com/n24q02m/mnemo-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
