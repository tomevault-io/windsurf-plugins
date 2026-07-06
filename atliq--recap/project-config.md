---
trigger: always_on
description: Guidance for AI coding agents - and humans - working in the RECAP repository. This is the **single source of truth** for how to set up, run, test, and change this project; tool-specific files (`CLAUDE.md`, etc.) import it. The human-facing overview is in [`README.md`](README.md); the security policy is in [`SECURITY.md`](SECURITY.md).
---

# AGENTS.md

Guidance for AI coding agents - and humans - working in the RECAP repository. This is the **single source of truth** for how to set up, run, test, and change this project; tool-specific files (`CLAUDE.md`, etc.) import it. The human-facing overview is in [`README.md`](README.md); the security policy is in [`SECURITY.md`](SECURITY.md).

## What RECAP is

RECAP is a **local-first** Chrome extension (Manifest V3) plus a local Python (FastAPI) RAG backend. It passively indexes the pages a user actually reads and lets them search their browsing history in natural language. Everything runs on the user's machine; page content only leaves the device when the user asks a question - and only the retrieved snippets, sent to the LLM provider the user configured.

## Setup & run

Requires **Docker** (recommended) or **Python 3.11+**, plus Chrome. Three paths, all of
which install spaCy + the `en_core_web_sm` model (so the knowledge graph works as soon as you opt in):

```bash
# Docker - recommended: reproducible, one command. Serves http://127.0.0.1:8000 (loopback only).
docker compose up --build

# uv - fast, reproducible local dev. `uv sync` installs deps + the spaCy model.
uv sync && uv run python main.py

# pip - simple fallback.
pip install -r requirements.txt && python main.py
```

`pyproject.toml` (+ `uv.lock`) is the canonical dependency set for uv/Docker; `requirements.txt`
mirrors it for the pip path - **keep the two in sync** when changing versions. torch is pinned to
the CPU wheel (`tool.uv.sources`) so images stay small and it runs anywhere; GPU users override it.

- Copy `.env.example` → `.env` and set at least one LLM key (or point at a local Ollama). See **LLM & embeddings** below.
- **Docker + a host Ollama:** the container can't see the host's `localhost` - set `LLM_BASE_URL`/`EMBEDDING_BASE_URL` to `http://host.docker.internal:11434/v1` (compose already maps `host.docker.internal`).
- Load the extension: `chrome://extensions` → enable **Developer mode** → **Load unpacked** → select the `extension/` folder.
- **Knowledge graph:** **off by default** - retrieval runs BM25 + dense vectors only. Opt in from the extension **Options page toggle** (calls `POST /settings/kg`; takes effect immediately, persisted in DB meta so it survives backend restarts and overrides `.env`), or set `ENABLE_KG=true` in `.env` for a config-only setup. The master switch gates both ingestion NER and the KG retrieval leg; a request's `use_kg` can never override it on. After enabling, backfill already-indexed pages without re-browsing via `POST /maintenance/rebuild_kg` (re-runs NER over stored text; SQLite is the source of truth). Entities need spaCy (installed by default).

## Testing

```bash
python tests/test_integration.py
```

Ten checks (config, models, SQLite/FTS5, LanceDB, knowledge graph, classifier, chunker, reranker, end-to-end pipeline, semantic gate). No pytest needed. On a Windows console, prefix `PYTHONUTF8=1` if you hit a Unicode error. **Run this before opening a PR and keep it green.**

## Architecture - the one rule that matters

**SQLite is the single source of truth; the keyword index and vector index are derived and reconstructible.**

- `data/recap.db` (SQLite) holds the canonical chunk **text** + all page/chunk metadata - stored exactly once.
- **FTS5** keyword search is an *external-content* table synced by triggers - it holds the inverted index only, no second copy of the text.
- **LanceDB** (`data/vector_store/`) stores only `chunk_id + vector` - no text, no metadata.
- Retrieval fuses BM25 + dense + KG with **weighted Reciprocal Rank Fusion** (k=60) → cross-encoder rerank → **time-decay recency** → then **hydrates** text/metadata from SQLite. Hydration is the single place chunk text is read and the single place the date filter is applied.
- Because the indexes are derived, changing the embedding model **rebuilds vectors from SQLite** (`POST /maintenance/reindex`) without losing any pages.

**Ingestion order:** `content_classifier` → semantic `chunker` (token-aware) → spaCy NER (optional; degrades gracefully) → SQLite → FTS (via triggers) → LanceDB → knowledge graph. Write SQLite (truth) **first**, derived indexes after, and commit the page's `content_hash` **last** - so a failure mid-pipeline re-indexes cleanly on the next visit instead of leaving a half-indexed page.

## Directory map

```
main.py                     entry point (uvicorn -> backend.app:app)
backend/
  app.py                    thin FastAPI layer - NO business logic here
  config.py                 pydantic-settings Settings (env / .env)
  models.py                 all Pydantic request/response models (bounded)
  bootstrap.py              schema-version clean-rebuild of local data/
  prompts.py                ALL LLM prompt templates + untrusted-content sanitizer
  embeddings.py             pluggable embedding fn (local OR OpenAI-compatible)
  ingestion/                processor, chunker, content_classifier, entity_extractor, semantic_gate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atliq/recap](https://github.com/atliq/recap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
