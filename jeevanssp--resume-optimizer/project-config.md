---
trigger: always_on
description: **ATS Beater** — AI-powered resume tailoring service by Cydratech. Upload PDF resume → AI structures it → paste a job description → AI tailors a custom resume → LaTeX compiles to PDF.
---

# CLAUDE.md

## What This Is

**ATS Beater** — AI-powered resume tailoring service by Cydratech. Upload PDF resume → AI structures it → paste a job description → AI tailors a custom resume → LaTeX compiles to PDF.

## Tech Stack

| Layer | Tech |
|-------|------|
| Backend | FastAPI, SQLAlchemy async, PostgreSQL 16 (Docker), Alembic |
| AI | `google-genai` SDK, **gemini-3-flash-preview** (profile/OCR), **gemini-3.1-pro-preview** (resume gen) |
| PDF | pdflatex + `resume.cls`, pdfplumber for text extraction |
| Frontend | Vue 3 + Tailwind + Pinia — all via CDN, no build step, hash router |
| Auth | Google OAuth 2.0 → JWT |
| Payments | Razorpay (credit packs: Day Pass ₹49, Sprint ₹99, Job Hunt ₹199) |
| Package mgr | UV, dependencies in `pyproject.toml` |

**Never use older 2.5 preview models — they are deprecated.**

## How To Run

```bash
docker compose up -d                          # PostgreSQL
uv sync --extra dev                           # install deps
uv run alembic upgrade head                   # migrations
uv run python -m app.main                     # server → http://localhost:8000
```

Set `DEV_AUTH_BYPASS=true` in `.env` to skip Google OAuth during development.

## How To Test

```bash
uv run pytest tests/ -v --ignore=tests/integration  # 99 unit tests (in-memory SQLite, no external deps)
INTEGRATION=1 uv run pytest tests/integration/ -v   # 9 smoke tests (needs real DB + API key + pdflatex)
```

## Data Flow

```
Upload PDF ──→ pdfplumber extract ──→ Gemini Flash structures ──→ ResumeInfo (JSON in PostgreSQL)
                                                                         │
Job Description ────────────────────────────────────────────────────────→ │
                                                                         ▼
                                                              Gemini Pro tailors
                                                                         │
                                                                         ▼
                                                              CustomResumeInfo
                                                                         │
                                                                         ▼
                                                         LaTeX builder + pdflatex ──→ PDF
```

Phase 1 and Phase 2 are separate API calls. User can review/edit `CustomResumeInfo` between phases.

## Credit System

**Business model**: Daily free quota + purchasable credit packs + unlimited time passes.

### Consumption Priority
1. **Active time pass** → unlimited (no deduction)
2. **Daily free** → 3/day (configurable via `DAILY_FREE_CREDITS`), resets at midnight UTC
3. **Purchased credits** → from balance
4. **No credits** → 429 error, frontend shows paywall modal

### Credit deduction
- Happens **synchronously** in the request handler (before background task starts)
- If background generation fails, a **refund** is issued (except for time pass — unlimited)
- All transactions recorded in `credit_transactions` audit ledger

### Promo codes
- Admin-created, types: `CREDITS` (adds N credits) or `TIME_PASS` (activates tier by ID)
- One redemption per user per code, optional max total redemptions, optional expiry

### Time pass stacking
- If a user buys a second pass while one is active, the new pass **starts at the old expiry**

## Job Status Flow

`PENDING` → `GENERATING_RESUME` → `RESUME_GENERATED` → `GENERATING_PDF` → `READY`

Any step can transition to `FAILED`.

## Project Layout

```
app/
  main.py                  # FastAPI factory, CORS, exception handlers, static file mount
  config.py                # Pydantic BaseSettings from .env
  dependencies.py          # get_current_user, get_super_admin, get_db
  exceptions.py            # Custom exception classes (→ HTTP 4xx handlers in main.py)
  models/                  # SQLAlchemy ORM (User, Profile, Job, Tenant, Credit*, etc.)
    __init__.py            # MUST import all models (relationship resolution)
    credit.py              # CreditPack, TimePassTier, UserCredit, UserTimePass, CreditTransaction, PromoCode, PromoRedemption
  schemas/
    credit.py              # All credit/payment Pydantic schemas
  database/session.py      # async engine, sessionmaker, get_db() generator
  services/
    ai/inference.py        # GeminiInference — structured output + retry
    ai/prompts.py          # System/user prompt templates
    ai/retry.py            # retry_decor with exponential backoff
    ocr/extractor.py       # PDFExtractor — pdfplumber first, Gemini vision fallback
    latex/builder.py       # CustomResumeInfo → LaTeX string
    latex/compiler.py      # LaTeX string → PDF bytes (60s timeout)
    latex/sanitizer.py     # Escape LaTeX special chars
    profile/service.py     # ProfileService — create, process (background), enhance, CRUD
    job/service.py         # JobService — generate_custom_resume (Phase 1), generate_pdf (Phase 2)
    credit/service.py      # CreditService — balance mgmt, check_and_deduct, promo, refund
    payment/razorpay_client.py  # RazorpayService — order creation, payment/webhook verification

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeevansSP/resume-optimizer](https://github.com/JeevansSP/resume-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
