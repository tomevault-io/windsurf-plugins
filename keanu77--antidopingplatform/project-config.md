---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

運動禁藥案例平台 — Full-stack MERN application for sports doping case database with educational content, data visualization, and TUE (Therapeutic Use Exemption) information. All UI in Traditional Chinese.

## Architecture

**Unified server pattern**: `server.js` at root serves both Express API (`/api/*`) and React SPA (Vite-built static files from `frontend/dist/`). Single process, single port.

- **Backend**: Express + MongoDB native driver. Mounted routes in `backend/routes/` are `casesFixed.js`, `statsFixed.js`, `education.js`, `tue.js` (see server.js:97-100). The two data routes (`casesFixed`/`statsFixed`) share a single `MongoClient` via `backend/db.js` (`connect()`/`getDb()`, in-flight-deduped), **separate** from server.js's Mongoose connection. DB name comes from `MONGODB_DB_NAME` (default `"sports-doping-db"`).
- **Frontend**: React 19 + Vite + React Router v6 + Tailwind CSS v3 + Chart.js
- **Database**: MongoDB, single collection `cases`. `server.js` opens a separate Mongoose connection (for lifecycle/auto-reconnect + registering `backend/models/Case.js` so its indexes get built), but request handlers query through the shared `MongoClient` in `backend/db.js` (`getDb()`), never `Case.find()`.
- **API client**: `frontend/src/services/api.js` — `import.meta.env.PROD ? "/api" : "http://localhost:${VITE_API_PORT||8080}/api"`. The frontend always calls the real API; there is **no mock data layer** (`mockData.js` was removed in 7eab93e).
- **Static content**: `education.js` serves WADA categories/quizzes/specialties from JSON in `backend/data/` (wada-categories.json, quizzes.json, medical-specialties.json). TUE page content (criteria, application guide, retroactive/lifecycle rules, disease cards, decision-tool scenarios) lives in the frontend at `frontend/src/data/tueGuide.js` and `tueDiseases.js` (rendered by `frontend/src/components/tue/*`); the old inline `tueContent` API was removed. The **drug substance list is the single source of truth in `backend/data/substances.json`** (structured schema: `wadaCode` / `prohibition` / `routes` / thresholds / `sportRestricted` / `tueEligible` / `washout`), read by both `/api/tue/check` and `/api/tue/substances`. The old inline `wadaSubstances` object was removed in the P1-C refactor.

### Key Data Model

`backend/models/Case.js` — WADA substance categories (S0-S9, M1-M3, P1), punishment details (banDuration, resultsCancelled, medalStripped), source links. Text index on athleteName/substance/sport/summary/nationality. Compound indexes on (sport, year), (nationality, year), (substanceCategory, year).

### Frontend Pages

Routes are defined in `frontend/src/App.jsx`. Only `Home`, `CaseList`, `CaseDetail` are eagerly imported; **all others are `React.lazy()`-loaded**.

- `Home` (`/`) — stats cards + recent cases
- `CaseList` (`/cases`) — multi-condition filtering (sport/nationality/year/substance/punishment) + text search + pagination
- `CaseDetail` (`/cases/:id`) — single case view
- `Statistics` (`/statistics`) — Chart.js visualizations (sport/substance/ban distribution, yearly trends)
- `Education` (`/education`) — WADA substance guide + interactive quizzes
- `TUE` (`/tue`) — thin tab shell; tabs are `components/tue/{Basic,Application,Diseases,Tools}Tab.jsx`. Tools tab: drug checker (calls `/api/tue/check`) and multi-step decision tool with lecture scenarios (route × in/out-of-competition × sport, powered by pure-function engine `frontend/src/utils/tueDecision.js`)
- Terminology: use 運動禁藥管制／防制 (not 反禁藥); S1 合成代謝劑, S5 利尿劑與掩蔽劑, S9 糖皮質激素. `scripts/rebuild-dataset.mjs` `unifyCategory` normalizes case-category spellings.
- `ProhibitedList` (`/prohibited-list`) — reads local `frontend/src/data/prohibitedList.js` (not the API)
- `Quiz` (`/quiz`) — reads local `frontend/src/data/quiz.js` (not the API)
- `TestingProcess` (`/testing-process`), `News` (`/news`), `Resources` (`/resources`) — mostly static content pages
- `NotFound` (`*`) — 404 catch-all

Shared: `components/Layout.jsx` (nav/footer wrapper), `components/ErrorBoundary.jsx` (wraps the whole `Router`). Client utils in `frontend/src/utils/` (`cache.js`, `debounce.js`).

### API Routes

| Route | Source | Notes |
|-------|--------|-------|
| `/api/cases` | `casesFixed.js` | Filtering, pagination, text search via regex |
| `/api/cases/filters` | `casesFixed.js` | Distinct values for filter dropdowns |
| `/api/stats/*` | `statsFixed.js` | MongoDB aggregation pipelines (yearly-trends, sport/substance/country/ban distributions) |
| `/api/education/*` | `education.js` | Reads from JSON files, quiz answers via POST |
| `/api/tue/*` | `tue.js` | `GET /tue/substances` (structured drug list from `substances.json`) + POST `/tue/check` drug eligibility checker (50 drugs, alias-aware, returns structured `prohibition`/`routes`/`tueEligible` fields) |
| `/api/health` | `server.js` | Health check (used by zeabur.json) |

## Commands

```bash
npm run install:all          # Install root + backend + frontend deps
npm run dev                  # Backend dev server (nodemon, port 8080)
npm run dev:frontend         # Vite dev server (port 5173)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keanu77/antidopingplatform](https://github.com/keanu77/antidopingplatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
