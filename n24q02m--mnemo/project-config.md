---
trigger: always_on
description: MCP Server cho AI memory. Python 3.13, uv, hatchling, src layout.
---

# AGENTS.md - mnemo

MCP Server cho AI memory. Python 3.13, uv, hatchling, src layout.
Post-de-host (2026-09): server la MOT HTTP Streamable endpoint tren hull-core
(auth `no-auth|token|multi` + per-namespace SQLite), local SQLite (WAL) la
storage authority duy nhat. Khong con stdio transport, Cloudflare deploy,
passport sync, relay setup, hay multi-provider key dispatch.
Hybrid search: FTS5 + sqlite-vec. 13 tools: 11 specialized memory tools
(add_memory, search_memory, list_memories, update_memory, delete_memory,
export_memories, import_memories, memory_stats, restore_memory,
archived_memories, consolidate_memories) + legacy `memory` dispatcher
(DEPRECATED -- use the granular tools instead) + config. 18 memory actions.
Provider calls di qua hull per-task `[models.*]` cells (base_url + api_key +
model, OpenAI-spec HTTP; OpenRouter pre-wired default):
`[models.embed]` (cloud embed; local Qwen3 ONNX fallback),
`[models.rerank]` (cloud rerank; local cross-encoder fallback),
`[models.chat]` (compression, entity extraction, consolidation),
`[models.jev_score]` (importance scoring). Cell khong co key => feature do
skip gracefully, KHONG fallback sang provider khac.

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
uv run mnemo-mcp                    # HTTP server (bind tu ~/.mnemo/config.toml)
uv run mnemo-mcp config-init        # ghi template config
uv run mnemo-mcp warmup             # pre-download local ONNX embed model

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
src/mnemo/
  __main__.py      # python -m mnemo entrypoint
  cli.py           # mnemo-mcp entry: bare = HTTP server; subcommands
                   # (config-init, warmup, token-hash, token-verify)
  server.py        # FastMCP tools/resources/prompts + hull HTTP app
  runtime.py       # bridge sang hull-core: settings, model cells, auth,
                   # per-namespace DB paths
  llm.py           # completion dispatch qua [models.chat] cell
  capture.py       # typed capture pipeline (dedup + compression hook)
  compression.py   # LLM compression qua chat cell ("chat-cell" marker)
  db.py            # SQLite: CRUD, FTS5, vector search (sqlite-vec)
  embedder.py      # [models.embed] cell + fastretrieval local fallback
  reranker.py      # local Qwen3 cross-encoder + [models.rerank] cell chain
  graph.py         # entity/relation extraction qua chat cell
  temporal/        # bitemporal KG: extract, resolve, store, queries
  alembic/         # schema migrations (mem_001..mem_003)
  setup_tool.py    # warmup logic
  providers.py     # bounded paid reflect provider (cap USD)
  pilot_tools.py   # CLI-surface handlers over mnemo_core
  docs/            # tool documentation markdown (memory.md, config.md)
src/mnemo_core/    # domain core for the `mnemo` CLI surface:
                   # operations, standing pages, ports, results, defense
src/mnemo_cli/     # `mnemo` / `mnemo-pilot` argparse CLI (no server)
tests/             # 1:1 mapping voi source modules
```

## Env vars

- `MNEMO_DB_PATH` / `DB_PATH` -- default `~/.mnemo/memories.db`
- `MNEMO_HOST` / `MNEMO_PORT` -- HTTP bind overrides (config.toml `[server]`)
- `MNEMO_AUTH_TOKEN` -- token source cho `mnemo-mcp token-hash`
- `HULL_<TASK>_API_KEY` -- host-only provider key override per cell:
  `HULL_EMBED_API_KEY` / `HULL_RERANK_API_KEY` / `HULL_CHAT_API_KEY` /
  `HULL_JEV_SCORE_API_KEY` (env wins over config.toml `api_key`)
- `EMBEDDING_DIMS` -- storage width (0 = runtime default 1024)
- `REINDEX_ON_MODEL_CHANGE` -- clear stale vector state on model swap
- `DISABLE_LOCAL_EMBED` / `DISABLE_LOCAL_RERANK` -- kill local ONNX fallback
- `LOCAL_EMBEDDING_MODEL` / `LOCAL_RERANK_MODEL` -- local model override
- `RERANK_ENABLED`, `RERANK_TOP_N` (10)
- `ARCHIVE_ENABLED`, `ARCHIVE_AFTER_DAYS` (90),
  `ARCHIVE_IMPORTANCE_THRESHOLD` (0.3), `ARCHIVE_TRIGGER_EVERY` (100)
- `DEDUP_THRESHOLD` (0.9), `DEDUP_WARN_THRESHOLD` (0.7)
- `RECENCY_HALF_LIFE_DAYS` (7)
- `COMPRESSION_ENABLED` (true)
- `KG_AUTO_ENABLED` (false), `TEMPORAL_ENTITY_RESOLUTION_THRESHOLD` (0.85),
  `TEMPORAL_SUPERSESSION_THRESHOLD` (0.85),
  `TEMPORAL_SUPERSESSION_ENABLED` (true)
- `MNEMO_REFLECT_MODEL`, `MNEMO_REFLECT_CAP_USD` -- paid reflect bounds
- `FASTRETRIEVAL_CACHE_PATH`, `LOG_LEVEL` (INFO)
- Removed 2026-09 (gone from code, do not reintroduce): `EMBEDDING_MODELS`,
  `RERANK_MODELS`, `LLM_MODELS`, `*_API_BASE` endpoints, per-provider
  `*_API_KEY` (JINA/GEMINI/OPENAI/COHERE/XAI/ANTHROPIC/VERTEX_EXPRESS),
  `COMPRESSION_PROVIDER`, `COMPRESSION_MODEL`, `SYNC_*`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n24q02m/mnemo](https://github.com/n24q02m/mnemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
