---
trigger: always_on
description: PaperVault is a fully-automated web service that collects, curates and searches AI / ML research-paper **metadata** from top-tier academic conferences and journals. It aggregates ACL Anthology, OpenReview, CVF Open Access, NeurIPS Proceedings, DBLP and other sources into a single, continuously-updated corpus and exposes it both as a downloadable dataset (on Hugging Face) and as a ready-to-use search website. The exact, machine-rendered coverage and corpus scale live in `README.md` (the `<!-- rec
---

# PaperVault - Agent Development Guide

## Project Overview

PaperVault is a fully-automated web service that collects, curates and searches AI / ML research-paper **metadata** from top-tier academic conferences and journals. It aggregates ACL Anthology, OpenReview, CVF Open Access, NeurIPS Proceedings, DBLP and other sources into a single, continuously-updated corpus and exposes it both as a downloadable dataset (on Hugging Face) and as a ready-to-use search website. The exact, machine-rendered coverage and corpus scale live in `README.md` (the `<!-- recent-update-* -->` / `<!-- stats-* -->` blocks rebuilt by `maintain.py`); do **not** hard-code these numbers in this document.

The backend has been refactored into a dedicated **`papervault/` Python package** built around a Flask application factory (`papervault.create_app`) and a versioned **`/api/v1/*`** REST surface. The legacy unversioned endpoints (`/api/search`, `/api/get_guess_you_like`) have been removed; the design notes for this refactor live in `docs/refactor-plan.md`.

This project was originally forked from [MLNLP-World/AI-Paper-Collector](https://github.com/MLNLP-World/AI-Paper-Collector) and is now developed independently under the name **PaperVault**.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Backend** | Python 3.8+ (CI uses 3.10), Flask 3.x, Werkzeug 3.x, **application factory** (`papervault.create_app`), **Pydantic v2** request/response schemas (`papervault/schemas.py`), unified JSON error envelope (`papervault/errors.py`), request-id structured logging (`papervault/logging.py`); `Settings` is a plain `@dataclass(frozen=True)` driven by env vars (`papervault/config.py`) — **not** `pydantic-settings` |
| **Frontend** | Vue 3.5 (Composition API + `<script setup>`), TypeScript 5.9, Vite 8, Vue Router 4 (hash mode), in-house **i18n** layer (`src/utils/i18n.ts`), WoS-style **query DSL** parser/evaluator (`src/utils/queryDsl.ts` + `src/utils/fields.ts`) powering both Smart Search and the dedicated Advanced Search view |
| **UI Framework** | Element Plus 2.14 (auto-imported via `unplugin-vue-components`), `@vueuse/core` |
| **HTTP Client** | Axios 1.x |
| **Data Collection** | BeautifulSoup4, Requests, PyYAML, tqdm, thefuzz / python-Levenshtein |
| **Data Artifacts** | huggingface_hub (dataset upload with `parent_commit` optimistic locking) |
| **AI Features** | Multi-provider LLM catalog (OpenAI, DeepSeek, Anthropic Claude, Qwen/DashScope, GLM, StepFun, custom) for keyword suggestion (`POST /api/v1/suggest`) and LLM-driven result re-ranking (`POST /api/v1/ai/rerank`). Wire formats: `openai-compatible` (via `openai` SDK) and `anthropic` (via `anthropic` SDK). Presets exposed through `GET /api/v1/ai/providers`; per-request overrides for provider / base_url / model / api_key. `tiktoken` for token counting |
| **Stats / Visualization** | numpy, matplotlib, wordcloud |
| **Build Tool** | Vite 8 with `vite-plugin-compression2` (gzip), `unplugin-auto-import`, `unplugin-vue-components` (auto-generated `auto-imports.d.ts` / `components.d.ts`) |
| **Tests** | Backend: `pytest` under `tests/` (`pytest.ini` → `testpaths=tests`); Frontend: zero-dependency `node:test` regression suite for the DSL parser (`web-vue/src/utils/__tests__/queryDsl.test.mjs`) |
| **Automation** | Playwright (used by `scripts/capture_screenshot.py` to regenerate the README hero screenshot) |

## Project Structure

```
PaperVault/
├── app.py                        # Thin Flask entrypoint: builds `app = create_app(settings)`; no business logic here
├── collector/                    # Multi-source data collector package for paper metadata
│   ├── __init__.py               # Re-exports full legacy public API (HEADERS, SESSION, collect, do_collect, load_cache, save_cache, all search_from_* etc.) for backwards-compat
│   ├── __main__.py               # `python -m collector` entrypoint (equivalent to the legacy `python collector.py`)
│   ├── http.py                   # Shared requests.Session singleton + HEADERS (User-Agent) + urllib3 Retry policy; also suppresses BS4 XMLParsedAsHTMLWarning
│   ├── merge.py                  # Field-level record merge helpers (_better_str/_list/_code, _merge_paper_record, _merge_with_cache)
│   ├── progress.py               # COLLECT_PROGRESS_FILE / COLLECT_FAILURES_FILE constants + load/save_collect_progress
│   ├── io.py                     # cache/cache.jsonl.gz reader/writer (load_cache, save_cache, _to_gz_path)
│   ├── code_links.py             # GitHub link extraction from abstracts (_GITHUB_RE, extract_github_link, add_code_links)
│   ├── pipeline.py               # Top-level orchestration: collect() drives SOURCE_REGISTRY over all sources; do_collect() wraps cache load/save + HF sync
│   └── sources/                  # Per-vendor scrapers (one module per data source)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youngfish42/PaperVault](https://github.com/youngfish42/PaperVault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
