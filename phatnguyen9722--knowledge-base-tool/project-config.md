---
trigger: always_on
description: A multi-feature, file-backed web application for managing personal knowledge:
---

# Knowledge Base Tool — Project Guide

A multi-feature, file-backed web application for managing personal knowledge:
files, documentation, training, and utilities. FastAPI + Jinja2 server-rendered
pages, Markdown files as the source of truth, SQLite only for indexes.

---

## Getting Started

Docker (recommended — matches production):
```bash
docker compose -f dockerise/docker-compose.yml up --build -d
```
The app is served at [http://localhost:5050](http://localhost:5050).

Local dev:
```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python cli.py serve            # http://127.0.0.1:5050
```

**Python 3.10+ is required.** Several modules use PEP 604 annotations
(`Transcript | None`) without `from __future__ import annotations`, so 3.9
raises `TypeError` at import. The Docker image pins `python:3.12-slim`.

---

## Architecture

```
app/
├── main.py          # ALL FastAPI routes (~2.4k lines) + module-level manager singletons
├── config.py        # load_settings() → Settings dataclass; every path comes from here
├── models.py        # Pydantic Post schema
├── parser.py        # frontmatter ↔ Post
├── post_manager.py  # Post CRUD + slugify + auto-index
├── search.py        # SQLite FTS5 engine (posts)
├── tag_manager.py   # tag counts + index.json
├── markdown.py      # render_with_toc()
├── exporter.py      # static HTML site export
├── hooks.py         # plugin event system (on_post_created/updated/deleted)
├── books.py         # collections, chapters, imported resource books (PDF/EPUB/…)
├── diagrams.py      # Mermaid/PlantUML sources + markdown export
├── notes.py  tasks.py  emails.py  bookmarks.py  api_docs.py  music.py
├── toeic.py  transcripts.py  vault.py  dictionary_db.py  sync.py
templates/           # Jinja2 — base.html + one set per feature
static/              # style.css, app.js, diagrams.js, vendored jszip/epub/mermaid
tests/               # pytest suite
cli.py               # Typer CLI (serve, new, list, search, build-index, export)
launcher.py          # pystray desktop tray launcher
config.yaml          # data dir + UI + server config
```

**Core patterns:**

- **One manager class per feature.** `NoteManager`, `TaskManager`, `BookManager`,
  … each owns a directory and does its own file I/O. Managers are instantiated
  **once at import time** in `app/main.py` (`pm`, `tm`, `books`, `notes`,
  `tasks_mgr`, `email_mgr`, `transcripts_mgr`, `dict_db`, …). `vault_manager` is
  a singleton created inside `app/vault.py` itself.
- **Markdown + YAML frontmatter is the source of truth.** SQLite (`.kb/`) holds
  only rebuildable indexes: `search.db` (FTS5) and `dictionary.db`.
- **Paths always come from `Settings`** (`app/config.py`), derived from
  `config.yaml`. Never hardcode a data directory.
- **Routes are thin.** Parse the form, call the manager, redirect `303`. Domain
  logic belongs in the manager module, not in `main.py`.
- Some routes use module-level path constants (`NOTES_DIR`, `POSTS_DIR`,
  `BG_DIR`, `FONTS_DIR`, `RESUME_PATH`, `LANG_PATH`) rather than the manager —
  relevant when testing (see below).

---

## Testing

```bash
python -m pytest tests/ -q                                   # full suite
python -m pytest tests/test_tasks.py -q                      # one file
python -m pytest tests/ -q --cov=app --cov=cli --cov-report=term-missing
```

Current state: **593 tests, 96% coverage** of `app/` + `cli.py`. Keep it there —
every new feature or bugfix lands with tests.

- Tests live in `tests/test_<feature>.py`. Manager unit tests first, then route
  tests through `TestClient`.
- PyMuPDF-dependent tests (PDF page rendering) skip automatically when `fitz`
  is unavailable, so a plain `requirements.txt` install stays green.
- **Use the `write-unit-test` skill** when adding tests — it documents the
  fixture patterns and the module-level-singleton monkeypatch gotchas.

---

## Feature Overview

All 12 major features in the system:

### 1. 📝 Posts
* **Main Function**: Article and long-form note publisher.
* **Short Description**: Allows creating and editing markdown posts. Supports tag organization, categorization, image uploads, and SQLite-backed FTS5 full-text search.

### 2. 📚 Series
* **Main Function**: Multi-part article binder.
* **Short Description**: Chains multiple standalone posts into structured series, creating next/previous page pagination and ordering for chapter-by-chapter reading.

### 3. 📖 Books & Reader
* **Main Function**: Digital library manager & immersive reader.
* **Short Description**: Organize book collections of chapters. Features an uploaded files section supporting **PDF, EPUB, MOBI, CBZ, FB2, XPS** formats. Offers a realistic **3D Page-Flip Reader** (spread-view for PDFs/Comics, responsive typography + TOC sidebar for EPUBs/MOBI), page jump, and resource deletion.

### 4. 🎧 TOEIC Test Preparation
* **Main Function**: Interactive practice sets for TOEIC training.
* **Short Description**: Features custom-formatted listening and reading tests. Users can select answers via radio inputs, play synced audio transcripts, submit answers for grade calculation, and view comprehensive explanation logs.

### 5. 🎵 Music Manager
* **Main Function**: Personal audio player and library organizer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phatnguyen9722/knowledge_base_tool](https://github.com/phatnguyen9722/knowledge_base_tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
