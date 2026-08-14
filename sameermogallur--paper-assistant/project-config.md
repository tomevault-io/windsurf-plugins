---
trigger: always_on
description: AIRA analyzes research papers for integrity and reproducibility signals. Upload a PDF and it extracts text natively via pypdf (OCR via Tesseract + Poppler is available as a separate `/api/parse_pdf_ocr` endpoint, not an automatic fallback on the ingest path), verifies every citation against the Crossref database with confidence scoring, extracts statistical indicators (p-values, sample sizes, effect sizes, confidence intervals), detects in-text citation markers, and returns a structured `Integri
---

# AIRA — AI Research Assistant

AIRA analyzes research papers for integrity and reproducibility signals. Upload a PDF and it extracts text natively via pypdf (OCR via Tesseract + Poppler is available as a separate `/api/parse_pdf_ocr` endpoint, not an automatic fallback on the ingest path), verifies every citation against the Crossref database with confidence scoring, extracts statistical indicators (p-values, sample sizes, effect sizes, confidence intervals), detects in-text citation markers, and returns a structured `IntegrityReport` with a heuristic integrity score (0–100, letter-grade A–F) based on open-science transparency markers.

> **Product strategy:** the reframed product vision, market-research question list, UI-redesign scope, and rescoped roadmap live in [VISION.md](VISION.md) (2026-07-22). This file stays focused on technical architecture.

## Architecture

```
User → React frontend (Vite)   port 5173 ──┐
User → Streamlit app_v2.py     port 8501 ──┼──▶ FastAPI app/ package   port 8000 ──▶ Crossref API
                                           │      via backend.py (3-line shim)   ──▶ OpenAlex API
                                           └───────────────────────────────────────▶ SQLite data/aira.db + PDFs data/pdfs/
```

### Key files

| File | Role |
|------|------|
| `README.md` | Public-facing project README (description, features, architecture, setup, API reference, testing, design decisions, limitations) — the front door for external readers |
| `backend.py` | 3-line shim — `from app.main import app`; entry point for uvicorn and CI |
| `app/` | FastAPI package — `main.py` (lifespan + CORS), `config.py` (env vars + constants), `routers/` (health, pdf, analysis, papers, projects), `services/` (pdf, references, citations, statistics, integrity, embeddings, ingest), `schemas/models.py`, `utils/helpers.py`, `db/` (engine + 6 ORM models) |
| `app/db/models.py` | SQLAlchemy ORM: Paper, AnalysisReport, Reference, Project, ProjectPaper, Embedding |
| `frontend/` | React + Vite + Tailwind + shadcn/ui — primary UI |
| `app_v2.py` | Legacy Streamlit frontend — calls FastAPI backend; still functional, secondary path |
| `app_legacy.py` | Original standalone Streamlit app (pre-API architecture, OpenAI-dependent, no Crossref, no OCR). Superseded by `app_v2.py` and the React frontend; kept for reference. |

## Prerequisites (one-time)

```bash
# macOS
brew install tesseract poppler

# Ubuntu/Debian
sudo apt-get install tesseract-ocr poppler-utils
```

## Local Development

### Venv setup

```bash
python3.11 -m venv .venv --clear
source .venv/bin/activate
pip install -r requirements.lock.txt        # reproducible pinned install
# (or `pip install -r requirements.txt` for a loose latest-compatible install)
```

Lockfiles (`requirements.lock.txt`, `requirements-dev.lock.txt`) are compiled with pip-tools and are what CI installs. After editing `requirements*.txt`, regenerate:

```bash
pip-compile --strip-extras --no-header -o requirements.lock.txt requirements.txt
pip-compile --strip-extras --no-header -o requirements-dev.lock.txt requirements.txt requirements-dev.txt
```

### Backend

```bash
source .venv/bin/activate
uvicorn backend:app --reload --port 8000
# Verify: curl http://localhost:8000/healthz
```

### React frontend (primary)

```bash
cd frontend
npm install
npm run dev        # → http://localhost:5173
```

### Streamlit frontend (legacy)

```bash
source .venv/bin/activate
streamlit run app_v2.py    # → http://localhost:8501
```

### Tests

```bash
pip install -r requirements-dev.txt
pytest tests/ -v
```

### Docker (backend only)

```bash
docker compose up --build
# First start downloads the SPECTER model (~500 MB) into ./data/hf-cache;
# DB, PDFs, and model cache all persist via the single ./data bind mount.
# Build is CPU-torch pinned to the lockfile version (see Dockerfile note).
```

## Environment Variables

Copy `.env.example` to `.env`:

| Variable | Required | Default | Notes |
|----------|----------|---------|-------|
| `CROSSREF_EMAIL` | **Yes** | `test@example.com` | Sent in Crossref User-Agent header for API politeness |
| `OPENAI_API_KEY` | No | — | AI summary in `app_v2.py` only; `backend.py` does not use it |
| `MAX_PDF_SIZE_MB` | No | `10` | |
| `MAX_PDF_PAGES` | No | `100` | |
| `MAX_OCR_PAGES` | No | `20` | |
| `CROSSREF_CONCURRENT_REQUESTS` | No | `3` | Semaphore limit for concurrent Crossref calls |
| `OPENALEX_MAILTO` | No | `CROSSREF_EMAIL` | Contact email for OpenAlex polite pool |
| `OPENALEX_CONCURRENT_REQUESTS` | No | `3` | Semaphore limit for concurrent OpenAlex calls |
| `POPPLER_PATH` | No | — | Windows only: path to poppler `bin/` directory |
| `ALLOWED_ORIGINS` | No | localhost 5173/3000/8501 | Comma-separated CORS origins; `*` allowed but disables credentials (browsers reject wildcard+credentials) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sameermogallur/paper-assistant](https://github.com/sameermogallur/paper-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
