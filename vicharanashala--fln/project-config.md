---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

FLN Assessment & Personalized Worksheet Platform — an AI-driven system that assesses each child's foundational **Mathematics** numeracy level (Classes 2–4), generates level-personalized printable worksheets, ingests scanned answers, evaluates them, and rolls data up a 7-role national hierarchy. See [SRS.md](SRS.md) (authoritative spec), [PRD.md](PRD.md) (product framing), [AUDIT.md](AUDIT.md) (current code health), and [MIGRATION_PLAN.md](MIGRATION_PLAN.md) (target structure).

**Stack:** MERN-ish — React 19 + Vite + Tailwind (frontend), Node/Express + TypeScript (backend), Python (AI evaluation pipeline), Google Gemini (LLM). The repo is an **npm-workspaces monorepo**: `frontend/`, `backend/`, `ai-services/` (see Layout).

## ⚠ Critical thing to understand before editing

**The app uses the real backend only. No mock backend should run.**

- The source of truth for all `/api/*` calls is the Express server at `backend/` (+ `ai-services/` Python pipeline). It implements the SRS: generation locks, defaulter escalation, Aadhaar masking, role-scoping, real Gemini, real PDF generation. It boots on `:3000` and is verified to answer the API directly via curl.
- The legacy in-browser mock (`frontend/src/mock/fetchInterceptor.ts`, `frontend/src/mock/dbStore.ts`) and the `setupFetchInterceptor()` call that previously lived at `frontend/src/main.tsx:8` are **deleted**. The frontend must talk to the real backend — no fallback, no parallel store.
- Any leftover `frontend/src/mock/**` files, the `public/mock/*.json` dataset, the `frontend/src/constants.ts` hardcoded seed, and `frontend/src/utils/levelGenerator.ts` (a byte-identical duplicate of `backend/src/levelGenerator.ts`) are slated for deletion — never reference them in new code.
- When asked to change "backend behavior," edit `backend/src/**` only. Do not reintroduce a second copy of business logic in the frontend.

See AUDIT.md for the full cleanup list and MIGRATION_PLAN.md for the deletion sequence.

## Layout

```
fln/                          # npm-workspaces monorepo root (package.json = workspaces)
├── frontend/                 # @fln/frontend — React + Vite app (talks to real backend on :3000 via proxy)
│   ├── index.html  vite.config.ts  tsconfig.json  package.json
│   ├── public/worksheets/    # worksheet HTML templates — ALSO read by the backend (Puppeteer)
│   └── src/
│       ├── main.tsx          # React entry; NO fetch interceptor
│       ├── App.tsx           # top-level views + role switch
│       ├── mock/             # deleted (was: fetchInterceptor.ts, dbStore.ts) — do not recreate
│       ├── constants.ts      # 763 ln of hardcoded seed data — slated for deletion; do not extend
│       ├── utils/levelGenerator.ts   # byte-identical duplicate of backend/src/levelGenerator.ts — slated for deletion
│       └── components/       # 24 components; RoleDashboards.tsx (2702 ln) + PanelViews.tsx (1455 ln) are god-files
├── backend/                  # @fln/backend — REAL Node/Express API (API only; no Vite)
│   ├── package.json  tsconfig.json  .env.example
│   ├── data/db.json          # the JSON-file "database" (not MongoDB despite comments)
│   └── src/                  # index.ts, db.ts, gemini.ts, paperGenerator.ts, ...
├── ai-services/              # REAL Python evaluation pipeline (run_pipeline.py, scripts/0..3, prompts/, syllabus/)
└── docs/                     # teacher workflow docs — describe the SERVER's behavior
```

## Commands

Run from the repo root (npm workspaces). One install covers both packages:

```bash
npm install
npm run dev:backend    # tsx backend/src/index.ts — real API on :3000  (REQUIRED; start this first)
npm run dev:frontend   # Vite dev server on :5173, proxies /api -> http://localhost:3000
npm run build          # builds frontend (vite) then backend (esbuild -> backend/dist/server.cjs)
npm run lint           # tsc --noEmit across workspaces (type-check only; there are no unit tests)
```

The app you see is the **frontend on :5173** talking to the **real backend on :3000**. Start the backend first; the frontend's Vite proxy (`vite.config.ts`) forwards `/api/*` to it. There is no in-browser mock — never add one. In production the backend serves `frontend/dist` (`FRONTEND_DIST_DIR`).

Demo login (e.g. `gps-mt-001.t01@fln.org`): **ask the team for the demo password** (do not hardcode or paste it into docs/commits). The Python pipeline needs `python` on PATH; the backend invokes it from `ai-services/` (`AI_SERVICES_DIR` override).

## Environment

- `GEMINI_API_KEY` — required for real AI calls (`backend/src/gemini.ts:9`). Each AI path has a deterministic non-AI fallback, so the server runs without it.
- `PORT` (default 3000), `CHROME_EXECUTABLE_PATH` (Puppeteer PDF generation).
- `AI_SERVICES_DIR` (defaults to `../ai-services`), `WORKSHEET_ASSETS_DIR` (defaults to `../frontend/public/worksheets`), `FRONTEND_DIST_DIR` (prod static serve).
- Copy `backend/.env.example`. Never commit real keys.

## Conventions & gotchas

- **Match the surrounding file's style** — this codebase was AI-generated by non-devs; consistency varies. Don't reformat wholesale.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vicharanashala/fln](https://github.com/vicharanashala/fln) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
