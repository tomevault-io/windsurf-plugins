---
trigger: always_on
description: ChassisForge is an AI-powered parametric robot chassis design tool for mechanical engineers. It accelerates the early conceptual design phase by generating structured design briefs, enabling rapid exploration of parametric design spaces, and providing real-time physics-based scoring — all backed by a curated database of real, purchasable components.
---

# ChassisForge — AI-Powered Robot Chassis Design Explorer

## Project Overview

ChassisForge is an AI-powered parametric robot chassis design tool for mechanical engineers. It accelerates the early conceptual design phase by generating structured design briefs, enabling rapid exploration of parametric design spaces, and providing real-time physics-based scoring — all backed by a curated database of real, purchasable components.

This is NOT a chassis generator that replaces MEs. It's a "first draft" tool that does the tedious constraint-checking homework so MEs can start CAD work with validated configurations instead of blank screens.

### Core Value Proposition

- Describe a task → get a structured design brief with real component recommendations
- Interactively explore parametric chassis configurations with sliders
- See real-time physics scores (stability, runtime, turning radius, payload capacity) update as you adjust
- Export a validated starting point: STEP file, BOM, URDF, design brief PDF

### Target User

Mechanical engineers, robotics startup teams, and advanced hobbyists designing wheeled/tracked mobile robot platforms for indoor logistics, inspection, delivery, agriculture, and similar applications.

-----

## Architecture

### System Diagram

```
┌─────────────────────────────────────────────────────┐
│                    Frontend (React)                  │
│  ┌─────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ Task Intake  │ │  Parametric  │ │  Scoring     │  │
│  │ (Chat/Form) │ │  Explorer    │ │  Dashboard   │  │
│  │             │ │  (Three.js)  │ │  (Recharts)  │  │
│  └──────┬──────┘ └──────┬───────┘ └──────┬───────┘  │
│         │               │                │           │
│         └───────────────┼────────────────┘           │
│                         │ REST API                   │
└─────────────────────────┼───────────────────────────┘
                          │
┌─────────────────────────┼───────────────────────────┐
│                  Backend (FastAPI)                    │
│  ┌──────────────┐ ┌────────────┐ ┌───────────────┐  │
│  │ Brief Gen    │ │ Parametric │ │ Simulation    │  │
│  │ (LLM + DB)  │ │ Geometry   │ │ Engine        │  │
│  │             │ │ (CadQuery) │ │ (analytic +   │  │
│  │             │ │            │ │  PyBullet)    │  │
│  └──────┬──────┘ └─────┬──────┘ └──────┬────────┘  │
│         │              │               │             │
│         └──────────────┼───────────────┘             │
│                        │                             │
│              ┌─────────┴─────────┐                   │
│              │ Component Database│                   │
│              │ (SQLite)          │                   │
│              └───────────────────┘                   │
└─────────────────────────────────────────────────────┘
```

### Tech Stack

- **Frontend**: React 18 + TypeScript, Three.js (via @react-three/fiber + @react-three/drei), Recharts for scoring dashboard, Tailwind CSS
- **Backend**: Python 3.11+, FastAPI, CadQuery for parametric CAD, PyBullet for advanced sim (Phase 4)
- **Database**: SQLite for component catalog
- **LLM**: Anthropic Claude API for design brief generation (Phase 2+)
- **Export**: CadQuery → STEP/STL, ReportLab or WeasyPrint → PDF, custom → URDF
- **Deployment**: Vercel (all phases)

### Deployment (Vercel)

Deploy to Vercel from day one. This covers all phases without migration.

**Phase 1 (client-side only)**:

- Vite React app deployed as static site to Vercel
- No backend needed — all physics runs in the browser
- Repo structure: monorepo with `frontend/` directory
- `vercel.json` at repo root points to `frontend/` as the build target
- Auto-deploys on push to `main`, preview deploys on PRs

**Phase 2+ (frontend + backend)**:

- Frontend remains on Vercel as before
- Backend (FastAPI) deployed as Vercel Serverless Functions via `api/` directory, OR as a separate service on Railway/Fly.io if serverless doesn't fit (CadQuery and PyBullet have heavy dependencies)
- If using separate backend host: configure CORS and set `NEXT_PUBLIC_API_URL` env var in Vercel
- SQLite works for dev; for production with a separate backend, use Turso (libSQL) or Postgres on Railway

**Repo structure**:

```
chassisforge/
├── vercel.json              # Vercel project config
├── CLAUDE.md                # This file
├── frontend/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── index.html
│   └── src/
│       └── ...
├── backend/                 # Phase 2+
│   ├── requirements.txt
│   ├── main.py
│   └── ...
└── api/                     # Vercel serverless functions (Phase 2+ alternative)
    └── ...
```

**`vercel.json`** (Phase 1):

```json
{
  "buildCommand": "cd frontend && npm run build",
  "outputDirectory": "frontend/dist",
  "framework": "vite"
}
```

**GitHub Actions CI** (optional but recommended):

- Lint + type-check on every PR
- Run physics engine unit tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
