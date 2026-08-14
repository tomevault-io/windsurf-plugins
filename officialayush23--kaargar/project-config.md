---
trigger: always_on
description: Kaargar is a dual-mode (Instant + Discovery) hyperlocal services marketplace for Pune, built on
---

# KAARGAR — Claude Code Build Context

Kaargar is a dual-mode (Instant + Discovery) hyperlocal services marketplace for Pune, built on
FastAPI + Supabase Postgres + React/Vite. The system is **already built and running** — this file
is a quick orientation and a set of hard rules for anyone (human or Claude) editing this repo.
For how the system actually works today (auth, matching engine, booking flows, job lifecycle,
penalties, admin config, gaps), see **`SYSTEM_OVERVIEW.md`** in this same directory. Read that
before making non-trivial changes — do not re-derive behavior from old specs.

Two older docs, `CLAUDE_LEGACY.md` and `KAARGAR_SYSTEM_DESIGN_v2_LEGACY.md`, are kept for history
only. They describe an OTP-based auth flow and a Cloudinary storage backend that were never built
this way, and other since-changed decisions. **Do not use them as a source of truth** — they are
archived, not current.

---

## CRITICAL RULES (unchanged, still enforced)

1. Frontend: **React + Vite + JSX only** — no TypeScript, no `.tsx` files.
2. Backend: **all SQLAlchemy models live in `BACKEND/models.py`**, **all Pydantic schemas live in
   `BACKEND/schemas.py`** — no exceptions, no `models/` or `schemas/` packages.
3. Media uploads: **Supabase Storage only.** No Cloudinary, no S3. There are now **six** buckets
   in active use (see `BACKEND/services/storage.py`), not the original two — see below.
4. Auth: **Supabase email+password**, validated on the backend as a Supabase-issued JWT. There is
   **no OTP send/verify flow** and the backend **does not issue its own JWT** — see
   `SYSTEM_OVERVIEW.md` for the full picture and why `APP_SECRET_KEY`/`jwt_secret_key` in
   `config.py` is present but currently unused.
5. Real-time: **Supabase Realtime** `postgres_changes` subscriptions on the frontend (no polling
   except the backend's own 500ms dispatch-acceptance poll during instant matching).
6. Background jobs: **APScheduler inside the FastAPI process** (`BACKEND/main.py` lifespan) — no
   Celery, no external job queue.
7. Redis: used for the **instant-dispatch lock** (`services/matching.py`), a **translation cache**
   (`services/translation.py`), and a lookup in `routers/workers.py`. It fails open (skips the
   lock) if unreachable — it is not a hard dependency. It is *not* currently used for OTP-send
   rate limiting (there is no OTP flow to rate-limit).
8. Payments: **Razorpay mandatory** — no cash option. Commission/GST is computed server-side in
   `services/matching.calc_commission` and deducted from the worker's payout (not added as a
   customer surcharge). Rates and thresholds are DB-config-driven via `platform_config`, with
   hardcoded fallback defaults — see `SYSTEM_OVERVIEW.md`.
9. Notifications: **SMTP-style email (via Resend) + Supabase Realtime in-app** — no push/FCM.
10. Discovery bookings always pin a specific worker chosen by the customer. There is **no**
    "auto-match" / "let the system find someone" path for Discovery — that was deliberately
    removed. Only Instant jobs use the expanding-radius matching engine.

---

## REPOSITORY STRUCTURE (actual, verified)

```
Kaargar/
├── CLAUDE.md                        ← this file
├── SYSTEM_OVERVIEW.md               ← comprehensive current-state doc, read this next
├── CLAUDE_LEGACY.md                 ← archived, stale, do not use as source of truth
├── KAARGAR_SYSTEM_DESIGN_v2_LEGACY.md ← archived, stale, do not use as source of truth
├── README.md, TODO.md, IMPLEMENTATION_AUDIT.md, JOB_COMPLETION_FLOW_PLAN.md,
│   KAARGAR_BACKEND_UPDATE.md, Kaargar_UI_CrossCheck.md   ← various working/progress notes,
│                                                            not guaranteed current — verify
│                                                            against code before relying on them
├── DEPLOYMENT_TODO.md               ← actively-maintained checklist: Render Celery worker/beat
│                                        deploy steps, local Docker dev setup, mobile local-vs-prod
│                                        API URL setup, remaining roadmap (OTP/FCM/promo/restructure)
├── 001_initial_schema.sql … 017_worker_areas.sql  ← migrations, applied in order (013 has two
│   files — 013_backfill_instant_availability.sql and 013_service_catalog.sql — check both)
├── seed_data.sql
├── BACKEND/
│   ├── main.py                      ← FastAPI app, CORS, router mounting, APScheduler lifespan
│   ├── config.py                    ← pydantic-settings Settings (see env vars below)
│   ├── database.py
│   ├── models.py                    ← ALL SQLAlchemy models (43+, incl. DeviceToken/PromoCampaign/
│   │                                   PromoCampaignDelivery/ContentTranslation — added once
│   │                                   models.py was reconciled against the live DB)
│   ├── schemas.py                   ← ALL Pydantic schemas (1250+ lines)
│   ├── dependencies.py              ← get_current_user / require_worker / require_admin
│   ├── alembic.ini, alembic/{env.py, script.py.mako, versions/baseline_20260803.py}
│   │                                   ← real Alembic project (was in requirements.txt but never
│   │                                   wired up before). Live DB needs `alembic stamp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [officialayush23/Kaargar](https://github.com/officialayush23/Kaargar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
