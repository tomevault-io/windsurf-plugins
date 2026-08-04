---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**PaperlessBrain** — AI-powered frontend for [Paperless-ngx](https://docs.paperless-ngx.com/). Built with NiceGUI + FastAPI. Lets users chat with their document archive (invoices, contracts, letters) via Claude or Ollama LLMs, with full tool-call agentic loop.

## Running

```bash
# Local dev
python main.py
# App runs at http://0.0.0.0:8080

# Deploy to LXC server
./deploy.sh           # code only
./deploy.sh --data    # code + data/ directory, niemals selbst ausführen!
```

## Dependencies

Single source of truth: `pyproject.toml` (direct deps only, range pins;
name `paperless-brain`, version from `config/version.py` via hatchling).
The old `requirements*.txt` files are gone.

```bash
# GPU machine (dev)
pip install -e ".[crawl,i18n]"

# CPU-only (LXC/Docker) — +cpu torch wheel outranks the CUDA build on PyPI
pip install --extra-index-url https://download.pytorch.org/whl/cpu -e ".[crawl,i18n]"
playwright install chromium   # only needed with the [crawl] extra

# Docker
docker build -t paperless-brain .                    # full
docker build --build-arg LEAN=1 -t paperless-brain . # no crawl4ai/chromium
docker compose up -d
```

Extras: `[crawl]` = crawl4ai + headless Chromium (JS-heavy page fetching —
app degrades to trafilatura-only without it); `[i18n]` = pybabel workflow.

## Configuration

All settings via `.env` file (loaded by `config/settings.py` via pydantic-settings). Required keys:

- `APP_PATH` — absolute path to repo root (with trailing slash)
- `PAPERLESS_URL`, `PAPERLESS_SUPERUSER_TOKEN` — Paperless-ngx API
- `IGNORE_INBOX_TAG_AT_SYNC` — tag name to skip during sync
- `EMBEDDING_MODEL`, `CHROMA_PATH`, `CHROMA_COLLECTION`, `EXTRACTION_SIDECAR_PATH`, `THUMB_PATH`
- `OLLAMA_SERVER`, `OLLAMA_INGEST_MODEL` — *optional*, vision LLM for ingestion
  (both also settable in Settings > Processing, which wins). Chat and Werkbank
  never read them — those use per-user models with their own `base_url`.
  `OLLAMA_SERVER` additionally names the host the WoL/shutdown buttons control;
  without it that is inferred from the first local-lane model in the registry.
- `ANTHROPIC_API_KEY` (optional) — global fallback key; chat models are managed per user in the UI (Settings > AI Models, `services/model_registry.py`)
- `SEARXNG_HOST` — self-hosted SearXNG instance
- `STORAGE_SECRET` — NiceGUI session secret

## Architecture

### Data flow

1. **Sync** (`pipelines/paperless_db_sync.py`): compares Paperless-ngx doc list against ChromaDB; calls `ingest_document` for new docs, `delete_document` for removed ones.
2. **Ingest** (`pipelines/ingest.py`): downloads PDF → extracts pages as images via pypdfium2 → sends each page to Ollama vision model → stores extracted JSON sidecar + ChromaDB embeddings.
3. **Sidecar** (`services/sidecar_service.py`): per-document JSON files at `{EXTRACTION_SIDECAR_PATH}/{doc_id}.json` — contain full summary, page texts, actions/deadlines, cross-references, tables.
4. **Cross-ref index** (`services/cross_ref_index.py`): built from all sidecars at startup; maps invoice numbers / file references to document IDs for the `get_related_documents` tool.
5. **Chat** (`services/chat_service.py`): agentic loop with `MAX_ITERATIONS=16`. Two backends: `ClaudeChatBackend` (Anthropic API streaming) and `OllamaChatBackend` (httpx POST). Both emit the same `ChatEvent` union type. Tools are defined in `TOOL_DEFINITIONS` (list of dicts, Claude format); converted to Ollama format via `_to_ollama_tools()`.

### Key singletons (`services/clients.py`)

Module-level singletons created at import time — one `PaperlessClient`, one `ChromaClient` for documents, one `ChromaClient` for brain facts (collection `"brain"`), `BrainService`, `SidecarService`, `CrossRefIndex`, `ThumbnailService`, `OllamaVisionClient`.

`get_session_paperless()` returns a user-scoped `PaperlessClient` using the session token from NiceGUI storage (falls back to admin token outside browser context).

### Brain / long-term memory

`services/brain_service.py` + `BrainService` stores user facts in the `"brain"` ChromaDB collection. Per-user isolation via `user` metadata field; `common=True` facts visible to all users. The chat `search` tool automatically queries brain for hints and prepends them to search results.

### UI

- NiceGUI 3.x, pages in `app_ui/pages/` (each imports `page_layout` + `require_auth` from `app_ui/layout.py`)
- Pages: `login`, `dashboard`, `browser` (document grid), `chat`, `brain` (fact management), `settings`
- Dialogs: `app_ui/document_dialog.py`, `app_ui/cluster_dialog.py`
- **NiceGUI pattern**: use `@ui.refreshable` for dynamic content in plain columns. `QTabPanels`/`element.clear()` silently fails.
- Chat events stream via `AsyncGenerator[ChatEvent, None]` — UI iterates with `async for` and updates reactively.

### Chat tool results that trigger UI dialogs

Three tools don't execute server-side — they return a special event that the UI page (`app_ui/pages/chat.py`) intercepts to open a dialog:
- `trigger_docx_generation` → `DocxRequestEvent` → DIN-5008 letter dialog
- `create_email` → `EmailRequestEvent` → email template dialog

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vailsen/paperless-brain](https://github.com/vailsen/paperless-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
