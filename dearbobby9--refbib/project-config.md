---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**RefBib** — A lightweight web app that extracts all references from an academic PDF and outputs standard BibTeX entries. Core workflow: drag PDF → auto-extract references → select entries → export `.bib` file.

Target users: researchers writing LaTeX/Overleaf papers who need to batch-collect BibTeX from reference lists.

**GitHub:** https://github.com/DearBobby9/RefBib
**Live:** https://ref-bib.vercel.app (password-protected)

## Architecture

```
Frontend (Next.js + shadcn/ui + TailwindCSS)     Backend (Python FastAPI)
┌──────────────────────────┐                     ┌─────────────────────────────┐
│  PDF drag-drop upload    │                     │  1. GROBID: PDF → structured│
│  Reference list + select │  ── /api/extract ─▶ │     citation list            │
│  Filter / search / sort  │                     │  2. BibTeX lookup waterfall: │
│  .bib export / clipboard │                     │     DOI→CrossRef (primary)  │
└──────────────────────────┘                     │     Title→Semantic Scholar  │
                                                 │     Title→DBLP (CS papers)  │
                                                 │     Fallback: GROBID raw    │
                                                 │  3. Post-processing:        │
                                                 │     citation-key dedup       │
                                                 └─────────────────────────────┘
```

- **Frontend**: Next.js 16 (App Router) + shadcn/ui + TailwindCSS v4
- **Backend**: Python FastAPI + httpx + lxml
- **PDF parsing**: GROBID (Docker self-hosted or public instances), REST API
- **BibTeX sources**: CrossRef API, Semantic Scholar API, DBLP API (all free, rate-limited)

## Development Commands

```bash
# One-click start (both frontend + backend)
./start.sh          # macOS/Linux
start.bat           # Windows

# Backend (manual)
cd backend
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Frontend (manual)
cd frontend
npm install
npm run dev

# Tests
cd backend && .venv/bin/pytest       # backend tests (84 tests)
cd frontend && npx vitest run        # frontend tests (16 tests)
cd frontend && npm run build         # frontend type-check + build

# GROBID (local Docker, optional)
docker run --rm -p 8070:8070 grobid/grobid:0.8.2-crf
```

## Key File Locations

### Backend (`backend/`)
- `app/main.py` — FastAPI app entry, CORS, lifespan (httpx clients + rate limiters)
- `app/config.py` — Settings + GROBID_INSTANCES list (6 instances)
- `app/routers/auth.py` — `/api/verify-password`, `/api/auth/status` (hmac.compare_digest)
- `app/routers/health.py` — `/api/health`, `/api/grobid-instances`, `/api/grobid-instances/{id}/health`
- `app/routers/references.py` — `POST /api/extract` (PDF upload + validation + GROBID fallback chain), `POST /api/discovery/check`, `POST /api/resolve-doi`
- `app/models/api.py` — Pydantic models (DiscoveryReferenceInput, ResolveDoiRequest/Response with DOI regex + field length validators)
- `app/services/bibtex_assembler.py` — Waterfall orchestrator (CrossRef → S2 → DBLP → fallback)
- `app/services/discovery_service.py` — Unmatched discovery: probe CrossRef/S2/DBLP for availability (decoupled from match_status)
- `app/services/grobid_service.py` — GROBID API client with fallback chain
- `app/services/grobid_xml_parser.py` — TEI XML → ParsedReference
- `app/services/crossref_service.py` — CrossRef DOI/title lookup
- `app/services/semanticscholar_service.py` — Semantic Scholar title lookup
- `app/services/dblp_service.py` — DBLP title lookup
- `app/utils/bibtex_formatter.py` — Citation key generation + LaTeX-aware BibTeX escaping
- `app/utils/rate_limiter.py` — Token bucket rate limiter
- `app/utils/text_similarity.py` — Title fuzzy matching

### Frontend (`frontend/`)
- `src/app/page.tsx` — Main page (unified queue: upload → batch progress → batch summary, single file auto-expands)
- `src/app/workspace/page.tsx` — Workspace page (search/filter, dedup stats, source papers, conflict queue, analytics, export); WorkspaceEntryCard displays authors (first 3 + "et al.") and venue/year metadata
- `src/components/pdf-upload-zone.tsx` — Drag-and-drop PDF upload (supports multi-file, max 20)
- `src/components/reference-list.tsx` — Results display with select/filter + DOI resolution override
- `src/components/reference-item.tsx` — Individual reference card (title links, Scholar search for all statuses, fuzzy warnings, DOI resolve input)
- `src/components/batch-progress.tsx` — Processing progress with per-file status + "Add more" append button
- `src/components/batch-summary.tsx` — Results with Radix Collapsible accordion, resume/retry, auto-expand for single file, append button
- `src/components/conflict-resolver.tsx` — Interactive conflict merge/keep-both UI
- `src/components/bibtex-editor.tsx` — Dialog-based BibTeX override editor
- `src/components/workspace-analytics.tsx` — Recharts dashboard (year bar, venue bar, match pie, most-cited)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DearBobby9/RefBib](https://github.com/DearBobby9/RefBib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
