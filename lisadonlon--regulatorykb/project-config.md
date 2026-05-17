---
trigger: always_on
description: Medical device regulatory affairs knowledge base. Python 3.9+ · Click CLI · SQLite + ChromaDB · PDF extraction with OCR · regulatory intelligence pipeline · Telegram bot · APScheduler automation.
---

# CLAUDE.md — RegulatoryKB

Medical device regulatory affairs knowledge base. Python 3.9+ · Click CLI · SQLite + ChromaDB · PDF extraction with OCR · regulatory intelligence pipeline · Telegram bot · APScheduler automation.

## File Map

### Root

| File | Lines | Purpose |
|------|------:|---------|
| `pyproject.toml` | 116 | Build config, deps (incl. `bot` extras), tool settings (Black, Ruff, MyPy, pytest) |
| `.pre-commit-config.yaml` | 16 | Ruff + format + whitespace + YAML + large-file hooks |
| `setup.py` | 11 | Shim for `pip install -e .` |
| `config/config.yaml` | 346 | Runtime config: paths, doc types, jurisdictions, OCR, intelligence, telegram, scheduler |
| `REGULATORY_CONTEXT.md` | 84 | Domain focus: medical devices, filtering rules, exclusions |

### `scripts/regkb/` — Core Package

| File | Lines | Purpose |
|------|------:|---------|
| `cli.py` | 42 | CLI bootstrap: logging, `Click` root group, command registration |
| `services.py` | 29 | Shared service accessors (`get_db`, `get_importer`, `get_search_engine`, etc.) |
| `commands/core.py` | 275 | Core top-level commands: search/add/list/show/update/stats |
| `commands/lifecycle.py` | 579 | Lifecycle/ops commands: import/ingest/extract/reindex/diff/gaps/download/acquire/versions/web |
| `commands/intel.py` | 1015 | `regkb intel` command group and subcommands |
| `config.py` | 251 | Singleton config manager; loads YAML, validates/normalizes types + jurisdictions |
| `database.py` | 415 | SQLite manager with FTS5 full-text search, CRUD, import batch tracking |
| `search.py` | 249 | Dual search: SQLite FTS5 + ChromaDB semantic vector search |
| `importer.py` | 358 | Import workflow: dedup, validation, text extraction, batch audit |
| `extraction.py` | 239 | PDF→Markdown via pdfplumber; OCR fallback via pytesseract |
| `downloader.py` | 210 | HTTP document fetcher with URL validation and filename sanitization |
| `diff.py` | 360 | Document comparison: unified diff, HTML side-by-side, similarity stats, export (CSV/MD/HTML) |
| `version_tracker.py` | 396 | Version checking against `KNOWN_VERSIONS` dict; current/outdated/unknown |
| `version_diff.py` | 244 | Auto-detect prior versions of imported docs and generate diffs |
| `gap_analysis.py` | 261 | Compare KB against reference checklist; identifier matching + scoring |
| `reference_docs.py` | 946 | Curated checklist of essential regulatory documents by jurisdiction |
| `acquisition_list.py` | 862 | Acquisition URLs for missing docs by jurisdiction + priority |

### `scripts/regkb/web/` — FastAPI Web UI

| File | Lines | Purpose |
|------|------:|---------|
| `main.py` | 80 | FastAPI app with lifespan, dotenv, session middleware, route registration |
| `lifespan.py` | 100 | Lifespan context manager: starts/stops APScheduler + Telegram bot |
| `health.py` | 45 | `/health` endpoint: status, uptime, scheduler jobs, Telegram connected |
| `dependencies.py` | 55 | DI functions (get_db, get_search_engine), flash messages |
| `routes/search.py` | 125 | Search page with HTMX live results |
| `routes/browse.py` | 145 | Document list, detail view, PDF download, text view |
| `routes/documents.py` | 185 | Upload PDF (with validation), import from URL, folder import, metadata edit |
| `routes/diff.py` | 110 | Document comparison with side-by-side HTML diff, export (CSV/MD/HTML) |
| `routes/versions.py` | 47 | Version status dashboard (current/outdated/unknown) |
| `routes/gaps.py` | 140 | Gap analysis dashboard with jurisdiction drill-down, CSV export |
| `routes/intel.py` | 400 | Intelligence pipeline: pending queue, approve/reject, digests, fetch/sync, send digest, Telegram notifications |
| `routes/admin.py` | 288 | Statistics, settings, backup, reindex, batch operations (re-extract, metadata) |
| `templates/` | 18 files | Jinja2: base, search, browse, detail, add, diff, versions, gaps, intel, batch, stats, settings + partials |
| `static/` | 3 files | Pico CSS, custom.css, htmx.min.js |

### `scripts/regkb/telegram/` — Telegram Bot Interface

| File | Lines | Purpose |
|------|------:|---------|
| `bot.py` | 75 | Bot factory: 7 commands + CallbackQueryHandler + MessageHandler for plain text |
| `auth.py` | 40 | `require_auth` decorator checking `TELEGRAM_AUTHORIZED_USERS` env var |
| `handlers.py` | 250 | Command handlers: /start, /help, /status, /digest, /pending, /search |
| `formatters.py` | 180 | MarkdownV2 escaping, entry/stats/search result formatting |
| `keyboards.py` | 100 | Inline keyboard builders: approve/reject/page/digest/search actions |
| `callbacks.py` | 200 | Callback query handlers: approve/reject/page/digest/search routing |
| `notifications.py` | 80 | Push notifications: critical alerts, job failures, digest sent |
| `search_handler.py` | 220 | Enhanced search: NL query parsing, jurisdiction aliases, pagination, follow-up |
| `llm_handler.py` | 170 | LLM Q&A: Nexa/Qwen NPU (simple) + Claude Haiku (complex), think-tag stripping |

### `scripts/regkb/scheduler/` — APScheduler Automation

| File | Lines | Purpose |
|------|------:|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lisadonlon/RegulatoryKB](https://github.com/lisadonlon/RegulatoryKB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
