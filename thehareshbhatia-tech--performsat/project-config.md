---
trigger: always_on
description: SEVA (formerly PerformSAT) is a digital-SAT prep web app: students take adaptive practice tests, get a diagnostic-driven study plan, drill weak skills, and track score trajectory toward a target. Built on Create React App + Firebase. ~256 JS/JSX files in `src/` (~9.7MB).
---

# CLAUDE.md — SEVA orientation for Claude Code (and humans)

SEVA (formerly PerformSAT) is a digital-SAT prep web app: students take adaptive practice tests, get a diagnostic-driven study plan, drill weak skills, and track score trajectory toward a target. Built on Create React App + Firebase. ~256 JS/JSX files in `src/` (~9.7MB).

The product was renamed PerformSAT → SEVA on 2026-05-21 (briefly via Sura → Seva). The logo (2026-06-15) is all-caps **"SEVA"** in rounded Baloo 2 where the **S is the tri-color mark** (orange/purple/lime horizontal bands) and each remaining letter takes one brand color — **E orange, V purple, A lime**. `src/components/ui/Mark.jsx` is the tri-color S (a Baloo 2 "S" with a vertical gradient clipped to the glyph via `background-clip: text`); it doubles as the favicon and the collapsed-sidebar icon. `src/components/ui/Wordmark.jsx` is the full SEVA wordmark (Mark + colored E/V/A) and feeds every render site. The favicon/PWA/OG PNGs in `public/` are rendered from the mark/wordmark. In prose the brand name is still written **SEVA**. Internal identifiers (the `performsat:` log-scope prefix, `localStorage['performsat:logVerbose']`, the `PERFORMSAT_TEST_EMAIL`/`PERFORMSAT_TEST_PASSWORD` env vars, the `PerformSAT` React component name in `src/App.jsx`, the repo directory `~/PerformSAT/`) deliberately retain the old name to avoid coordinated-rename risk. New code may use either name in identifiers — be consistent within a module.

The codebase is mid-scale, mostly mature, with one large orchestrating file (`src/App.jsx`, ~2.5k lines) that owns view state and practice-session state. The recent direction is closing UX gaps vs Acely AI, surfacing the deeper diagnostic engine, and tightening drill routing to exact-question-type precision (see `docs/DRILL_ROUTING_PLAN.md`).

This file is the orientation document for new contributors and LLM agents. Keep it up to date when architecture moves.

## Quick start

```bash
npm install
npm start          # dev server (CRA, port 3000)
npm test           # Jest watcher
CI=true npx react-scripts test --watchAll=false   # run all tests once
```

Firebase config lives in `src/firebase/config.js`. To run against a real project you need a `.env.local` with the Firebase keys (template at `.env.local.template`). To dogfood the study plan or drill flow locally, run `npm run dev:emulator` then `npm run dev:seed` (seeds one student + a completed test + a study plan). No `schools` doc is needed: the school/principal (B2B2C) model was removed 2026-05-29 in favor of direct-to-consumer; the app is now owner-only.

## The big picture

```
Student logs in
   │
   ▼
StudentDashboard  ──────── Default landing. Two tabs with count badges:
   │                       "Dashboard (N)" + "Study Plan (N)"
   │   ├─ Dashboard tab → main column (TodaysTasksCard with per-activity
   │   │                  sub-cards + Predicted vs Actual) +
   │   │                  right rail (CalendarMonth + score-with-delta +
   │   │                  goal/exam two-up + DashboardDiagnosticWidget)
   │   └─ Study Plan tab → <StudyPlanDashboard /> with focus-area cards
   │                        (italic Diagnostic Sentence below each)
   │
   ▼  (student takes a practice test)
PracticeTest.jsx  ──────── Full-length test runner. Builds groundTruth diagnosis
                            on completion, persists studyPlanArtifact to Firestore.
   │
   ▼  (test completes)
diagnosticEngine.js  ──── Classifies errors (6-class taxonomy), aggregates skill
                            accuracy, builds weaknesses[]/strengths[]/etc.
   │
   ▼
studyPlanGenerator.js ── Generates week-by-week plan from the diagnosis.
   │
   ▼  (drill flow)
StudyPlanDashboard.jsx ── "Skills to Improve" cards. Each card calls
                            getTargetedWeaknessSet() with the weak skill →
                            returns drill question IDs.
   │
   ▼  (student clicks Practice)
AssignedPracticeShell.jsx ─ The PRODUCTION DRILL PATH. Renders the question +
                            feedback panel for assigned (study-plan-driven) drills.
```

## The three practice shells (important — pick the right one)

There are three components that look like "the drill UI." They serve different purposes:

| Shell | Purpose | Mount path | When to use |
|-------|---------|------------|-------------|
| `AssignedPracticeShell.jsx` | **Production drill path** for study-plan focus areas | `App.jsx` view='practice', `practiceMode='assigned'` | Anything the student reaches via "Practice" button on Study Plan |
| `AdaptivePracticeShell.jsx` | Alternate adaptive practice (difficulty adjusts) | `App.jsx` view='practice', `practiceMode='adaptive'` | The adaptive flow launched separately. Not study-plan-driven. |
| `PracticeTest.jsx` | Full-length timed practice test | `App.jsx` view='takingTest' | Only for full mock tests, not drills |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thehareshbhatia-tech/PerformSAT](https://github.com/thehareshbhatia-tech/PerformSAT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
