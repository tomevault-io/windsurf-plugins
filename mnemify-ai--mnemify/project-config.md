---
trigger: always_on
description: > Read this before touching any code. This file is a **map to the code**, not
---

# AGENTS.md — Mnemify Orientation

> Read this before touching any code. This file is a **map to the code**, not
> a substitute for it. It stays intentionally short; the code and its module
> docstrings are the source of truth. If anything here conflicts with the
> code, the code wins.

---

## Where things live

| Area | Path | Notes |
|---|---|---|
| Harvester (Tier 0) | `backend/src/harvester/` | One package per source (`notion/`, `confluence/`, `jira/`, `obsidian/`, `slack/`, `github/`, `_google/`). Plugin contract is `SourcePlugin` in `harvester/__init__.py`: `health_check()`, `list_documents(since)`, `fetch_document(ref)`, optional `mark_harvested()`; register with `register_plugin(name, factory)`. Only the packages in `ENABLED_SOURCES` are imported — see `src/sources.py`. |
| Terrain compiler (Tier 1) | `backend/src/terrain/` | File map below. |
| FastAPI + SSE (Tier 2) | `backend/src/api/` | `__init__.py` builds the app and mounts the built frontend (`paths.web_dist_dir()`); with no build it logs a WARNING and serves a "frontend not built — run `sh setup.sh`" page at `/` rather than booting API-only. Harvest/compile progress streams over SSE from an in-process event bus (`event_bus.py`, `compile_bus.py`) with a replay ring buffer for reconnecting tabs. Schedules (`/api/schedules`) run on an in-process APScheduler — single uvicorn worker only. |
| On-device embeddings | `backend/src/api/routes_embeddings.py`, `terrain/utils/local_embedder.py` | Claude engines with no `OPENAI_API_KEY`: `start_compile` refuses with `code="openai_key_missing"` + `local_embeddings_eligible`; the frontend `LocalEmbeddingsDialog` (mounted in `DashboardLayout`, reached through `useStartCompile`) explains the trade-off (runs locally, English-only, coarser regions), links to Settings, and on consent calls `POST /embeddings/local/prepare`, which downloads bge-small into `<home>/.mnemify/models` and saves it as the `embedding_model` default. The model never downloads silently — `LocalEmbeddingClient` loads with `local_files_only`. The compile dialog (`AiModePicker`) has an **Embeddings: OpenAI / On this computer** switch (OpenAI disabled without a key); a per-run pick that differs from the saved default is persisted by `start_compile` so Ask and schedules stay in the same vector space. With a key set but the on-device default never downloaded, the refusal is `local_model_missing` + `openai_key_set` and the dialog offers OpenAI instead of the download. |
| Chat (`/api/ask`) | `backend/src/api/routes_ask.py`, `ask_retrieval.py`, `ask_chunks.py`, `ask_expansion.py`, `ask_providers.py`, `ask_agent.py` | Flow: `understand_query()` → embed → chunk search over `terrain.db` vectors → graph walk/rerank → bundle (≤15 items) → expand to raw chunk text → SSE `retrieval_debug → citations → delta* → citations_used → done`. The chat-LLM key arrives per request in `Authorization: Bearer` and is never persisted; query embeddings use the model stamped on `graph_node_vectors` (OpenAI via the server's `OPENAI_API_KEY`, or the on-device model) so vectors match the compile. Frontend side: `frontend/web/src/ask/`. |
| CLI | `backend/src/cli.py` | `mnemify harvest / terrain build / up / stop / status / inspect / normalize / purge / debug / reset / migrate-home / login`. Run via `uv run mnemify …` from `backend/`. `up` is single-instance (an already-running server means "print the URL, open the browser, exit 0") and falls forward up to ten ports if `--port` is taken; `stop` POSTs the shutdown route using `<home>/server.port`; `migrate-home` moves a legacy `backend/` layout into the platform home. |
| On-disk paths | `backend/src/paths.py` | **Every** path the app persists to resolves here — `home()`, `data_dir()`, `yaml_file()`, `env_file()`, `logs_dir()`, `server_pid_file()`, `server_port_file()`, `web_dist_dir()`. Home precedence: `MNEMIFY_HOME` → legacy (`backend/` already holds `.mnemify/`, `mnemify.yaml` or `.env`) → platform app-data dir. `MNEMIFY_ENV_FILE` / `MNEMIFY_YAML_FILE` still win for those two files. |
| Shipped connectors | `backend/src/sources.py` | `ENABLED_SOURCES = ("notion", "confluence", "obsidian")` — what the release exposes. The other five plugins stay in the tree, tested, just never registered; `MNEMIFY_SOURCES=notion,jira` turns any subset back on for one process. Gates the registry's imports and `GET /api/connections`, not the plugin code. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mnemify-ai/mnemify](https://github.com/mnemify-ai/mnemify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
