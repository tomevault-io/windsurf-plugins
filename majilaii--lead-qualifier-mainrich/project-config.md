---
trigger: always_on
description: @.github/instructions/writingStye.instructions.md
---

# Hunt - Lead Qualifier

## Writing style
@.github/instructions/writingStye.instructions.md

## What this is
B2B lead discovery, qualification, and outreach platform. Built for Mainrich International (NdFeB magnet supplier) but industry-agnostic - ICP is defined dynamically per search. Full pipeline: chat-based ICP definition → Exa neural search → website crawling → LLM scoring → contact enrichment → deep research → AI-generated email sequences → HubSpot CRM sync. ~150 API endpoints, 20+ dashboard pages, Stripe billing.

## Architecture
- **Backend**: Python 3.12 / FastAPI, runs in Docker as `hunt-api` on port 8000. 14 route modules, 5 background task loops
- **Frontend**: Next.js 16 / React 19 / TypeScript / Tailwind v4, runs as `hunt-web` on port 3000. All custom Tailwind, no component library
- **Database**: Supabase (PostgreSQL), remote hosted
- **Auth**: Supabase Auth with JWT verification (JWKS)
- **LLMs**: Gemini (primary - chat, scoring, extraction, ICP parsing), Claude (email generation, deep research)
- **Search**: Exa AI neural search + Exa Websets (persistent monitored discovery)
- **Scraping**: Crawl4AI (primary) → Cloudflare Browser Rendering (fallback) → Playwright (last resort)
- **Email**: Gmail SMTP with open/click tracking, reply/bounce detection via IMAP
- **CRM**: HubSpot OAuth2 (Companies, Contacts, Deals)
- **Billing**: Stripe (checkout, subscriptions, webhooks, customer portal)
- **Enrichment**: RocketReach (primary) → PDL → Exa (cascade)
- **Notifications**: Resend (daily digest, alerts)
- **Tunnel**: Cloudflare Tunnel at hunt-api.mainrichinternational.com (for Exa webhooks)

## How to run
```bash
# Everything (backend + frontend with hot-reload):
docker compose up --build -d

# Backend only:
docker compose up backend

# Check logs:
docker logs hunt-api --tail 50

# Rebuild after dependency changes:
docker compose up --build -d
```

## Testing
```bash
# Backend tests (from backend/ dir, venv activated):
cd backend && pytest tests/ -x -q

# Run a single test file:
pytest tests/test_intelligence.py -x -q

# Promptfoo evals (email quality):
cd backend/promptfoo && npx promptfoo@latest eval
```
- pytest config: `asyncio_mode = auto`, testpaths = `tests/`
- Always run the specific test file you changed, not the whole suite
- Tests use conftest.py fixtures, check `tests/conftest.py` before writing new tests

## Deployment
- **Backend**: Railway (`hunt-api`). Production deploys from `main`, staging from `staging` branch.
- **Frontend**: Vercel. Production deploys from `main`, preview deploys from any non-main branch.
- **Workflow**: Push to `staging` → test on Railway staging + Vercel preview → PR to `main` → prod deploy.
- **Branch protection**: `main` requires PR, no direct pushes.

## Key backend files
- `chat_server.py` - FastAPI app, route registration, middleware, background task startup
- `routes/` - 14 route modules (chat, pipeline, leads, enrichment, sequences, dashboard, searches, schedules, websets, crm, observability, experiments, diagnostics, icp)
- `chat_engine.py` - Dual-LLM conversation engine, parameter extraction, query generation
- `pipeline_engine.py` - Main qualification pipeline (crawl → score → enrich → deep research)
- `intelligence.py` - LLM scoring with penalty adjustments, intent signal boost, ICP criteria evaluation
- `websets_engine.py` - Exa Websets integration (create, sync, process, webhooks)
- `email_generator.py` - Claude-powered cold email generation with writing style instructions
- `sequence_engine.py` - Multi-step email sequence processing loop (15 min interval)
- `email_sender.py` - Gmail SMTP with open/click tracking
- `scraper.py` - Website crawling (Crawl4AI, Cloudflare, Playwright fallback chain)
- `enrichment.py` - Contact enrichment via RocketReach (primary) + PDL fallback
- `deep_research.py` - Multi-page AI research + Exa Answer API
- `criterion_evaluator.py` - Custom ICP criteria evaluation (threshold, LLM classify, LLM assess, conditional)
- `scheduler.py` - Recurring pipeline schedules + daily requalification loop
- `auto_enroll.py` - Post-pipeline automatic sequence enrollment
- `hubspot_client.py` - HubSpot CRM sync (OAuth2, Companies/Contacts/Deals)
- `daily_digest.py` - Automated daily summary email (8am UTC)
- `cost_logger.py` - Per-call cost tracking for every LLM and API call
- `config.py` - All configuration, thresholds, API keys
- `db/models.py` - SQLAlchemy ORM models

## Key frontend paths
- `src/app/dashboard/` - 20+ dashboard pages (overview, leads, pipeline, map, websets, sequences, analytics, experiments, diagnostics, settings, etc.)
- `src/app/chat/` - Chat interface (ICP definition + live search)
- `src/app/components/chat/ChatInterface.tsx` - Core chat component (~2700 lines, handles full discovery flow)
- `src/app/components/chat/LiveMapPanel.tsx` - Mapbox GL live map with glowing dots
- `src/app/components/` - Shared components (navbar, footer, auth, billing, pipeline, UI primitives)

## Frontend conventions
- TypeScript, functional components only
- Tailwind v4 with custom design tokens (see `globals.css` for `--color-*` vars)
- Color classes: `text-text-primary`, `text-text-muted`, `text-text-dim`, `bg-surface-2`, `border-border`, `text-hot`, `text-review`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/majilaii) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
