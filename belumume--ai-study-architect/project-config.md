---
trigger: always_on
description: **Study Architect** by Quantelect — Mastery-based AI study companion that proves you learned it. CS education beachhead. Claude API (primary) + OpenAI (fallback).
---

# CLAUDE.md

## Project Overview

**Study Architect** by Quantelect — Mastery-based AI study companion that proves you learned it. CS education beachhead. Claude API (primary) + OpenAI (fallback).

**Live**: https://aistudyarchitect.com
**Design**: [design/DESIGN.md](design/DESIGN.md) (canonical tokens, typography, glow recipes)
**Strategy**: [docs/direction/NEW_DIRECTION_2025.md](docs/direction/NEW_DIRECTION_2025.md)
**Build plan**: [docs/plans/2026-03-14-001-feat-full-product-build-phases-neg1-0-1-plan.md](docs/plans/2026-03-14-001-feat-full-product-build-phases-neg1-0-1-plan.md)
**Brainstorm**: [docs/brainstorms/2026-03-13-mvp-frontend-brainstorm.md](docs/brainstorms/2026-03-13-mvp-frontend-brainstorm.md)

## Development Rules

### NO EMOJIS in Code or Terminal Output
- Windows terminals fail with Unicode emojis (UnicodeEncodeError)
- Use ASCII: [PASS], [FAIL], [WARN], SUCCESS:, ERROR:
- Exception: robot emoji in Claude attribution commit trailers

### Frontend Styling Rules
- **New components**: Tailwind v4 + shadcn/ui (Radix) + Lucide icons
- **Legacy components** (chat, content): MUI + Emotion — Phase 3 removal pending
- Design tokens from `@theme` in `src/index.css` — never hardcode hex colors
- `cn()` from `@/lib/utils` for conditional class merging
- Tailwind class order: sorted by prettier-plugin-tailwindcss
- `npx shadcn@latest add <component>` — add one at a time (batch silently fails)

## Common Commands

### Backend
```bash
cd backend
uvicorn app.main:app --reload                  # Dev server (port 8000)
pytest tests/ -v                                # Run tests
ruff check app/ --fix                          # Lint + fix
alembic upgrade head                           # Run migrations
alembic revision --autogenerate -m "message"  # New migration (needs local PG running)
```

### Frontend
```bash
cd frontend
npm run dev                                    # Dev server (port 5173)
npm test                                       # Vitest
npm run typecheck                             # TypeScript check
npm run lint                                  # ESLint
npm run build                                 # Production build (tsc && vite build)
# Tailwind v4: @tailwindcss/vite plugin, no tailwind.config needed
# shadcn/ui: npx shadcn@latest add <component>
```

### Quality Checks
```bash
# Backend full check
cd backend && ruff check app/ && mypy app/ && pytest tests/ -v

# Frontend full check
cd frontend && npm run lint && npm run typecheck && npm test

# E2E
cd frontend && npx playwright test
```

### Deploy
```bash
cd worker && npx wrangler deploy               # Backend (CF Container)
# Frontend: auto-deploys from GitHub via Vercel
```

## Architecture

### Stack
- **Frontend**: React 18 + TypeScript + Vite 6 + Tailwind v4 + shadcn/ui → Vercel
- **Backend**: FastAPI → Cloudflare Container (Docker, 1/4 vCPU, 1 GiB)
- **Worker**: CF Worker routes `/api/*` → Container, everything else → Vercel proxy
- **Database**: Neon PostgreSQL (serverless), Alembic migrations
- **Cache**: Upstash Redis (REST API, _NoOpCache fallback)
- **Storage**: Cloudflare R2 (file uploads + backups)
- **AI**: Claude API (streaming SSE) → OpenAI fallback. Direct SDKs, no LangChain.

### Backend Structure
```
backend/app/
├── api/v1/
│   ├── api.py              # Main router (12 sub-routers)
│   ├── auth.py, chat.py, tutor.py, content.py, concepts.py
│   ├── subjects.py         # Subject CRUD
│   ├── study_sessions.py   # Session lifecycle (start/pause/resume/stop)
│   ├── dashboard.py        # Dashboard summary (3-query pattern)
│   ├── admin.py, agents.py, csrf.py, websocket.py
│   └── endpoints/backup.py
├── agents/                  # Lead tutor (Socratic questioning)
├── core/                    # config, security, database, csrf, cache, rate_limiter, rsa_keys, utils
├── models/                  # user, content, study_session, subject, practice, chat_message, concept, user_concept_mastery
├── schemas/                 # Pydantic v2 schemas (UUID-based, model_config)
└── services/                # ai_service_manager, claude_service, openai_fallback, content_processor, concept_extraction, storage
```

### Frontend Structure
```
frontend/src/
├── app/layout/              # AppShell, TopNav (Tailwind)
├── pages/                   # DashboardPage, StudyPage, FocusPage, ContentPage
├── components/
│   ├── dashboard/           # HeroMetrics, SubjectList, ContributionHeatmap, StartFocusCTA (Tailwind + SVG)
│   ├── ui/                  # shadcn/ui primitives (button, card, dialog, dropdown-menu, input, tabs, badge, tooltip, sonner)
│   ├── auth/                # LoginForm, RegisterForm, ProtectedRoute (Tailwind + shadcn)
│   ├── chat/                # ChatInterface (MUI — legacy, Phase 3 restyle)
│   └── content/             # ContentUpload, ContentList, ContentSelector (MUI — legacy)
├── hooks/useTimer.ts        # Web Worker timer hook
├── workers/timer.worker.ts  # Date.now()-based background timer
├── contexts/AuthContext.tsx  # Global auth state
├── services/                # api.ts (Axios), auth.service.ts, tokenStorage.ts (legacy cleanup stub)
├── lib/utils.ts             # cn() helper (clsx + tailwind-merge)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/belumume) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
