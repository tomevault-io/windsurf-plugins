---
trigger: always_on
description: This file is written for AI coding agents. It assumes no prior knowledge of the project. Read it before modifying code, adding features, or debugging.
---

# AGENTS.md — SDPD Developer Guide

This file is written for AI coding agents. It assumes no prior knowledge of the project. Read it before modifying code, adding features, or debugging.

> **Important:** The top-level `README.md` is outdated. The codebase has grown well beyond the 33-case campaign described there. This file reflects the actual project state.

---

## 1. Project Overview

**SDPD (Systems Design Police Department)** is an interactive, gamified learning platform for distributed systems design. The primary audience is system-design interview preppers, but it is also usable for courses and self-paced learning.

The app is a single-page React application that runs almost entirely in the browser:

- **Detective Campaign** — 33 sequential failure cases. Inspect diagrams, diagnose root cause, prescribe a fix.
- **System Builder** — 23 drag-and-drop architecture-design challenges graded against concept rules.
- **Chaos Simulator** — 3 presets where players inject faults and apply fixes while watching metrics/logs.
- **Daily Drill** — One timed case per UTC day, same for everyone worldwide, with stars + streaks.
- **Mock Interview** — 3 timed one-attempt rounds plus a written postmortem with a scoring rubric.
- **Detective's Notebook** — Spaced-repetition review built from missed questions and saved key terms.
- **System Design Cheatsheet** — 45 decision cards, searchable and tabbed by category.
- **Optional accounts / leaderboard** — GitHub OAuth via Supabase for cross-device sync and daily-drill leaderboard. Fully playable without it.

All gameplay progress is stored in `localStorage` by default. Supabase is strictly opt-in.

---

## 2. Technology Stack

| Layer | Technology |
|-------|------------|
| Framework | React 19 (StrictMode) |
| Language | TypeScript 5.8 |
| Build tool | Vite 6.3 |
| Styling | Tailwind CSS 4.1 with custom theme (`src/index.css`) |
| Routing | React Router 6.30 (`react-router-dom`) |
| State | Zustand 5.0 |
| Diagrams | XYFlow (`@xyflow/react`) for case and builder canvases |
| Drag & drop | `@dnd-kit/core` (builder palette) |
| Animation | Framer Motion 12.34 |
| Fonts | Fontsource: Bebas Neue, DM Sans, Space Mono |
| Optional backend | Supabase (`@supabase/supabase-js`) |
| Testing | Vitest 4.1 + jsdom |
| Linting | ESLint 9.25 + `typescript-eslint` + `eslint-plugin-react-hooks` + `eslint-plugin-react-refresh` |

Node 20+ and npm 8+ are expected.

---

## 3. Project Structure

```
sdpd/
├── public/                    # Static assets (og-image, vite.svg)
├── scripts/
│   ├── generate-case-index.mjs         # Builds src/data/case-index*.json
│   ├── validate-builder-challenges.js  # Schema/parity check for builder JSONs
│   ├── randomize-answers.js            # One-off helper (not used in build)
│   └── randomize-answers.py            # One-off helper (not used in build)
├── src/
│   ├── components/
│   │   ├── account/           # Sign-in, handle picker, account section
│   │   ├── builder/           # Builder canvas, palette, report card
│   │   ├── chaos/             # Chaos diagram
│   │   ├── common/            # Button, Badge, ProgressBar
│   │   ├── diagram/           # React Flow nodes (database, server, client, builder)
│   │   ├── game/              # CaseBrief, DiagnosisPanel, SystemDiagram, etc.
│   │   ├── guide/             # Educational concept guide panel
│   │   ├── interview/         # Interview setup, round, postmortem, debrief
│   │   ├── layout/            # Header, Sidebar, GameLayout, MobileMenu
│   │   └── leaderboard/       # Daily leaderboard panel
│   ├── context/
│   │   └── CloudAccountContext.tsx
│   ├── data/
│   │   ├── builder/           # 23 EN + 23 pt-BR challenge JSONs
│   │   ├── cases/             # 33 EN case JSONs
│   │   │   └── pt-BR/         # 33 pt-BR case translations
│   │   ├── case-index.json    # Generated lightweight case list (EN)
│   │   ├── case-index.pt-BR.json
│   │   ├── categories.ts      # Case category ranges
│   │   ├── chaos/presets.ts   # 3 chaos simulator presets
│   │   ├── cheatsheet/        # en.json + pt-BR.json + parity test
│   │   ├── concepts.json      # Educational concept material (EN)
│   │   └── concepts-pt-BR.json
│   ├── engine/
│   │   ├── validator.ts       # Multiple-choice answer validation
│   │   └── builderGrader.ts   # Pure grading engine for builder designs
│   ├── hooks/
│   │   ├── useGameState.ts    # Main campaign progress store
│   │   ├── useBuilderChallenges.ts
│   │   ├── useBuilderProgress.ts
│   │   ├── useChaosSimulator.ts
│   │   ├── useCheatsheet.ts
│   │   ├── useCloudAuth.ts
│   │   ├── useCloudSync.ts
│   │   ├── useDailyDrill.ts
│   │   ├── useInterviewSession.ts
│   │   └── useNotebook.ts
│   ├── i18n/
│   │   ├── index.ts
│   │   ├── useTranslation.ts
│   │   └── locales/
│   │       ├── en.json
│   │       └── pt-BR.json
│   ├── lib/
│   │   ├── leaderboard.ts     # Supabase leaderboard queries
│   │   └── supabase.ts        # Optional Supabase client
│   ├── pages/
│   │   ├── HomePage.tsx
│   │   ├── CasePage.tsx
│   │   ├── BuilderListPage.tsx
│   │   ├── BuilderPage.tsx
│   │   ├── ChaosPage.tsx
│   │   ├── CheatsheetPage.tsx
│   │   ├── DailyDrillPage.tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olucasandrade/sdpd](https://github.com/olucasandrade/sdpd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
