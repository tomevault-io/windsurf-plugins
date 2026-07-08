---
trigger: always_on
description: A legal-data labelling app for producing training data for a legal LLM.
---

# LabelLex

A legal-data labelling app for producing training data for a legal LLM.
First concrete use case: upload bank prospectuses (typically 200–400 pages,
templated by magic-circle law firms) and label clauses for MREL eligibility
under the EU bank-resolution framework (BRRD/SRMR).

The app must show prospectuses with **original PDF layout intact** —
columns, indentation, footnotes — because legal layout carries semantic
meaning. The model loop (a local Ollama LLM) will eventually pre-label
clauses; humans correct; corrections feed the next training signal.

## Stack

- **Backend:** Python 3.11+, FastAPI, SQLAlchemy 2 (sync), SQLite (v0; Postgres later), pymupdf for PDF parsing.
- **Frontend:** React 18 + TypeScript + Vite, pdfjs-dist for PDF rendering.
- **LLM (planned):** local Ollama on `localhost:11434`, target hardware RTX 5070 Ti / 16 GB VRAM.

## Repo layout

```
backend/
  pyproject.toml
  app/
    main.py            FastAPI entrypoint (lifespan: create_all + migrations + seed)
    config.py          Settings (DB url, storage dir, default user/project, Ollama, LoRA Forge)
    db.py              Engine, SessionLocal, Base, get_db, lightweight ALTER-TABLE shim
    models.py          ORM models
    schemas.py         Pydantic request/response models
    seed.py            Idempotent seed: default user, project, starter labels
    services/
      pdf_parser.py        pymupdf → ParsedDocument (canonical text + bboxes)
      storage.py           File store for uploaded PDFs
      attributes.py        collect_effective_attributes + value-type validation
      label_counts.py      group-by helper for LabelOut.annotation_count
      document_activity.py touch_document + attach_annotation_counts (per-doc)
      ollama.py            sync httpx client, status() + generate_structured()
      structure_detector.py TOC outline → MREL section-type classification
      clause_discovery.py  per-page Ollama call → re-anchored verbatim quotes
      strategies/          Strategy interface + zero_shot impl + router stub
    routers/
      projects.py      /api/projects (CRUD)
      labels.py        /api/projects/:id/labels (CRUD)
      attributes.py    /api/labels/:id/attributes (CRUD)
      categories.py    /api/projects/:id/categories (CRUD)
      documents.py     Upload, list, get, PATCH (category), /pdf, /pages/:n
      annotations.py   POST/GET/PATCH/DELETE /api/annotations
      search.py        /api/documents/:id/search
      structure.py     /api/ollama/status, /api/documents/:id/detect-structure
      suggestions.py   /api/labels/:id/suggest-attributes,
                       /api/documents/:id/prelabel  (NDJSON stream),
                       /api/documents/:id/suggestions,
                       /api/suggestions/:id/accept|reject
      relations.py     /api/projects/:id/relation-defs (CRUD),
                       /api/relations (CRUD),
                       /api/documents/:id/relations (list)
      publish.py       /api/projects/:id/publish-to-lora-forge
                       (POST: bundle docs+annotations → sibling LoRA Forge)
  scripts/
    parse_pdf.py       Standalone parser test harness (run on a fixture PDF)

frontend/
  package.json
  vite.config.ts       Dev proxy: /api → http://127.0.0.1:8000
  src/
    main.tsx, App.tsx  App owns only the route table; nested layout in ProjectShell
    api.ts             Typed API client
    types.ts           Shared types (mirror backend schemas)
    pages/
      ProjectsListPage.tsx   /projects — list + create + delete projects
      ProjectShell.tsx       /projects/:id layout (sidebar + Outlet)
      ProjectPage.tsx        /projects/:id — drag-drop upload + document table
      ProjectSettingsPage.tsx /projects/:id/settings — document categories CRUD
      LabelsPage.tsx         /projects/:id/labels — labels admin
      DocumentViewer.tsx     /projects/:id/documents/:docId — PDF viewer
    components/
      PdfPage.tsx            pdf.js canvas + word-bbox overlay + label picker
      AnnotationListPanel.tsx Sticky right-side annotation list
    utils/
      spans.ts               lineRects, pageAnnotationSlice, effectiveAttributes
    styles.css
    vite-env.d.ts

storage/uploads/{id}.pdf       Uploaded PDFs (gitignored)
backend/labellex.db[-wal|-shm] SQLite + WAL sidecars (gitignored)
example_*.pdf                  Test fixtures (gitignored)
```

## Run

Backend (from repo root):

```
.venv\Scripts\python.exe -m uvicorn app.main:app --reload --app-dir backend --port 8000
```

Frontend:

```
cd frontend
npm install   # first time only
npm run dev   # opens on http://localhost:5173 (or 5174 if 5173 is taken)
```

Tables and the default user/project/labels are created on backend startup
(`lifespan` → `Base.metadata.create_all` → `seed`). Idempotent.

To re-parse the canonical fixture from the command line:

```
.venv\Scripts\python.exe backend\scripts\parse_pdf.py
```

## Architecture invariants

These are load-bearing decisions; check before reworking.

- **Backend is canonical for text + offsets.** pymupdf extracts per-word
  `(char_start, char_end, bbox, block, line)` and persists it. The frontend
  uses pdf.js for *visual rendering only* and pulls word data from
  `GET /api/documents/:id/pages/:n` so character offsets in annotations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RickGeurts/LabelLex](https://github.com/RickGeurts/LabelLex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
