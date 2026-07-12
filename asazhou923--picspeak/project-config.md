---
trigger: always_on
description: This file gives Claude Code project-specific guidance for working in this repository.
---

# CLAUDE.md

This file gives Claude Code project-specific guidance for working in this repository.

## Project Overview

PicSpeak is an AI photo critique and visual-reference generation web app. Users can upload photos for structured AI photography feedback, browse review history and gallery content, and generate AI reference images from prompts, templates, or review improvement suggestions.

Core product areas:

- Photo upload, AI critique, scoring, and retake suggestions
- Guest and authenticated usage with quotas and upgrade paths
- Public gallery, blog, changelog/updates, SEO metadata, and llms.txt support
- AI image generation with templates, tasks, generated image detail pages, history, credits, and credit-pack billing
- Review-to-generation loop for composition, lighting, color, and retake reference images
- Review-to-workspace retake targets, history practice themes, and in-task Blog reading during critique/generation waits
- Operational health snapshots for task status, AI costs, credits, payments, and public-content audits

## Architecture

- **Frontend**: Next.js 15 App Router, React 18, TypeScript, Tailwind CSS
- **Backend**: FastAPI, SQLAlchemy 2.x, Alembic, Uvicorn
- **Database**: PostgreSQL
- **Object storage**: Cloudflare R2 / S3-compatible storage
- **AI critique**: OpenAI-compatible vision LLM endpoints, with Flash and Pro model settings
- **AI generation**: OpenAI-compatible image generation endpoint, task queue, credit pricing, and object-storage persistence
- **Task processing**: In-process async worker by default, optional standalone worker and Cloud Tasks configuration
- **Authentication**: Clerk plus legacy Google OAuth/guest JWT support
- **Billing**: Lemon Squeezy Pro checkout, activation codes, image credit packs, and webhooks

## Common Commands

### Backend setup

```bash
python -m venv .venv
.\.venv\Scripts\activate
pip install -r backend/requirements.txt
```

### Backend runtime

```bash
cd backend
python scripts/ensure_runtime_schema.py
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Backend worker and maintenance

```bash
cd backend
python -m app.worker_main
python -m app.cleanup_guests_main
python scripts/generate_activation_codes.py
python scripts/register_lemonsqueezy_webhook.py
python scripts/verify_product_analytics_write.py
python scripts/export_product_analytics_weekly_report.py
python scripts/export_operational_health_snapshot.py
python scripts/backfill_gallery_thumbnails.py
```

### Backend tests

```bash
.\.venv\Scripts\python.exe -m pytest backend/tests
.\.venv\Scripts\python.exe -m pytest backend/tests/test_image_generation_routes.py
```

### Frontend setup and runtime

```bash
cd frontend
npm install
npm run dev
npm run dev:clean
npm run build
npm run start
```

### Frontend checks

```bash
cd frontend
npm run lint
npm run typecheck
npm run test
```

## Project Structure

### Backend (`backend/app/`)

- `main.py` - FastAPI app entry point, lifespan, middleware, exception handlers
- `api/router.py` - API router assembly under `/api/v1`; legacy webhook router under `/api`
- `api/routers/` - Domain routes for auth, uploads, photos, reviews, tasks, gallery, generations, blog, billing, analytics, realtime, and webhooks
- `api/deps.py` - Request dependencies for auth, quota, guest tokens, and shared route helpers
- `db/models.py` - SQLAlchemy models for users, photos, reviews, tasks, gallery, billing, usage, analytics, and generated images
- `db/bootstrap.py` - Runtime schema bootstrap helpers
- `services/ai.py` and `services/ai_prompts.py` - Vision critique client and prompt construction
- `services/review_task_processor.py` - Photo review task execution
- `services/image_generation*.py` - Generation client, prompt building, pricing, and task execution
- `services/object_storage.py` - Presigned upload/download and generated image persistence
- `services/task_dispatcher.py`, `services/task_events.py`, `services/worker.py` - Task dispatch, WebSocket/event polling, and worker orchestration
- `services/clerk_auth.py`, `services/clerk_webhooks.py` - Clerk identity and webhook handling
- `services/lemonsqueezy*.py` - Checkout, webhook, Pro, and credit-pack handling
- `services/product_analytics.py`, `services/content_audit.py`, and `services/operational_health.py` - Analytics, content conversion, and operational health reporting support
- `core/config.py` - Environment-backed settings via `pydantic-settings`

### Backend scripts and schema

- `backend/alembic/` - Alembic migrations
- `backend/scripts/ensure_runtime_schema.py` - Runtime schema guard used during local startup/deploys
- `create_schema.sql` - Full schema snapshot, useful for reviewing table/index intent

### Frontend (`frontend/src/`)

- `app/` - App Router routes, including workspace, reviews, tasks, gallery, generate, generation tasks/details, account pages, blog, updates, localized pages, robots, sitemap, and llms.txt routes
- `features/workspace/` - Upload flow, quota display, mode/image type pickers, and replay context
- `features/reviews/` - Review detail hooks and UI panels, including action bar, gallery publishing, growth loop, retake target handoff, and reference generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsaZhou923/picspeak](https://github.com/AsaZhou923/picspeak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
