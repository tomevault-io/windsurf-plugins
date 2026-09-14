---
trigger: always_on
description: Read [AGENTS.md](AGENTS.md) before working. It defines the canonical public
---

# CLAUDE.md

Read [AGENTS.md](AGENTS.md) before working. It defines the canonical public
repository, mandatory target checks and environment boundaries for every agent.
This file adds architecture and coding context; keep it consistent with AGENTS.md.

## Startup Checks

On load, verify required tools are installed:

```bash
# Check agent-browser (required for e2e testing)
command -v agent-browser >/dev/null || echo "WARNING: agent-browser not installed. Run: npm install -g agent-browser && agent-browser install"
```

## Project Overview

Facebook Ad Automation App - A full-stack application for automating the lifecycle of Facebook video and image ads, from competitor research to ad creation, launching, and performance reporting.

**Tech Stack:**
- Frontend: React 19 + Vite + TailwindCSS
- Backend: Python FastAPI (Python 3.11+)
- Database: PostgreSQL on Railway
- Storage: Persistent Railway media volume; optional Cloudflare R2 (S3-compatible)
- Testing: agent-browser (e2e), Vitest (unit)
- Hosting: Railway (backend + frontend + worker + PostgreSQL)

## Development Commands

Follow [local development](docs/deployment/local-development.md) with a dedicated
PostgreSQL database and process environment. The repaired `setup.sh` validates
configuration and runs the v2 bootstrap; Docker Compose provides all four local
services. For manual startup, run `python startup.py` from `backend/` and start
the worker with `python -m app.sync_worker`. `init_db.py` alone does not complete
v2 installation setup.

Frontend commands from `frontend/`: `npm ci`, `npm run dev`, `npm run test:unit`,
`npm run test:coverage`, and `npm run build`. Tests and servers require the target
checks and cleanup in AGENTS.md. Use the isolated database variables and commands
in [.github/workflows/test.yml](.github/workflows/test.yml) for backend validation.

## Architecture

### Database Models (backend/app/models.py)

Core entities and their relationships:

- **Brand**: Central entity with logo, colors (primary/secondary/highlight), voice
  - Has many Products (cascade delete)
  - Has many CustomerProfiles (many-to-many via brand_profiles table)
  - Has many GeneratedAds

- **Product**: Belongs to Brand, contains description, product_shots (JSON), default_url

- **CustomerProfile**: Demographics, pain_points, goals - linked to Brands

- **WinningAds**: Template library with structural analysis, blueprint_json for Ad Remix Engine

- **GeneratedAd**: Output from AI generation, links Brand + Product + Template, includes ad_bundle_id for grouping

- **FacebookCampaign/AdSet/Ad**: Hierarchy for Facebook campaign management with fb_*_id fields for syncing

- **ScrapedAd**: Competitor ads from research module

### Backend Structure (FastAPI)

```
backend/app/
├── main.py              # FastAPI app, CORS, router registration
├── database.py          # SQLAlchemy engine, SessionLocal, Base
├── models.py            # All SQLAlchemy models
├── core/
│   └── config.py        # Settings, validates DATABASE_URL is PostgreSQL
├── api/v1/              # API endpoints (all prefixed /api/v1)
│   ├── brands.py
│   ├── products.py
│   ├── profiles.py      # Customer profiles
│   ├── generated_ads.py # AI-generated ads
│   ├── facebook.py      # Campaign/AdSet/Ad management
│   ├── research.py      # Competitor scraping
│   ├── ad_remix.py      # Blueprint deconstruction/reconstruction
│   ├── copy_generation.py
│   ├── templates.py
│   ├── uploads.py
│   └── dashboard.py
├── services/            # Business logic
│   ├── facebook_service.py    # Facebook Marketing API (facebook-business SDK)
│   ├── research_service.py
│   ├── scraper.py
│   └── ad_remix_service.py    # Uses Gemini Vision for template analysis
└── schemas/             # Pydantic models (if exists)
```

**Key Backend Patterns:**
- All routes use `/api/v1` prefix
- Database dependency injection via `Depends(get_db)`
- PostgreSQL required - config.py validates DATABASE_URL on startup
- Facebook API uses `facebook-business` SDK (AdAccount, Campaign, AdSet, Ad, AdCreative, AdImage)
- AI services use Google Gemini (GEMINI_API_KEY) and Fal.ai (FAL_AI_API_KEY)
- File uploads go to Cloudflare R2 when configured, falls back to local `uploads/` for dev

### Frontend Structure (React + Vite)

```
frontend/src/
├── App.jsx              # Router setup, wraps with ToastProvider/BrandProvider/CampaignProvider
├── main.jsx             # Entry point
├── components/          # Reusable UI components
│   ├── Layout.jsx       # Main layout with navigation
│   ├── Toast.jsx        # Toast notification component
│   ├── Wizard.jsx       # Multi-step wizard
│   ├── BrandForm.jsx
│   ├── ProductForm.jsx
│   ├── CustomerProfileForm.jsx
│   └── ...wizard steps and builders
├── pages/               # Route components
│   ├── Dashboard.jsx
│   ├── Research.jsx     # Competitor analysis
│   ├── CreateAds.jsx    # Ad creation flow
│   ├── ImageAds.jsx
│   ├── VideoAds.jsx
│   ├── AdRemix.jsx      # Template remix engine
│   ├── GeneratedAds.jsx # View generated ads
│   ├── Brands.jsx
│   ├── Products.jsx
│   ├── CustomerProfiles.jsx
│   ├── FacebookCampaigns.jsx
│   ├── WinningAds.jsx   # Template library
│   └── Reporting.jsx
├── context/             # React Context for global state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonakatiff/theleadrouter-ad-studio](https://github.com/jasonakatiff/theleadrouter-ad-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
