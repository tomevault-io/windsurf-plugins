---
trigger: always_on
description: > Last updated: May 6, 2026 (Batch 61 — Machine chip persistence fix: v11 library backfill + inheritance promotion + gym-prefer pass-3 fallback)
---

# Gains — Project State

> Last updated: May 6, 2026 (Batch 61 — Machine chip persistence fix: v11 library backfill + inheritance promotion + gym-prefer pass-3 fallback)

## Rules for Claude

1. **Read this file first** at the start of every new session before doing anything else.
2. **Update this file** after every batch of changes. Add new features to "Recent Changes", update file structure if files were added/removed, and update store shape if state changed. Update the "Last updated" date.
3. **Git is fully writable from the sandbox.** Claude runs `git` directly — creates worktrees, commits, pushes feature branches, and merges to main. Never `--force` push. Never skip hooks (`--no-verify`).
4. **Validate builds** with `npx vite build --outDir /tmp/test-build`. Never build to the mounted `dist/` folder (Vite can't emit there — EPERM).
5. **Run `npm run lint` before every commit.** ESLint is wired with `react-hooks/rules-of-hooks` + `no-undef` as errors — these catch the two bug classes that crashed the app on April 26 (dangling identifier ref + hook after early return). Exit code must be 0 before merging. `exhaustive-deps` and `no-unused-vars` stay as warnings (visible but non-blocking).
6. **Feature branches for non-trivial changes.** Not for review (user doesn't review), but to give a clean revert point and a Vercel preview URL before merging to main. Small fixes can go straight to main.

## Pre-flight checklist for redesign batches

The April 26 crash bugs (`missedYesterdayWorkout` dangling ref + `CreateExerciseModal` hooks-after-early-return) both slipped past ~12 polish rounds because verification was happy-path-only — the dev loop only exercised one split type and one modal-open state. Run this checklist before merging any redesign batch:

- [ ] `npm run lint` → exit 0 (errors block; warnings are fine).
- [ ] `npx vite build --outDir /tmp/test-build` → success.
- [ ] Walk the redesigned surface in preview against THREE state combinations:
  - [ ] **Empty data** (`sessions: []`, no splits beyond built-in).
  - [ ] **Populated weight-only data** (BamBam's Blueprint active, sessions logged).
  - [ ] **HYROX-active data** (HYROX Hybrid active, with or without sessions).
- [ ] For surfaces with rotation/calendar logic: test today=rest AND today=workout (HYROX Hybrid Sunday=rest is a quick toggle).
- [ ] For surfaces with modals/sheets: open it, close it, re-open it. Watch console for "Rendered more hooks than during the previous render."
- [ ] Open browser console → verify zero new errors AND zero new warnings.
- [ ] If the surface reads `theme.hex` for foreground colors: test daylight + a light accent (the pale-green case is the known-bad combination).

The top-level `ErrorBoundary` (April 27) catches any render error and shows a recovery banner with the stack trace + Reload + recovery-page link, so the worst-case future bug is contained — but the checklist still matters because the boundary is recovery, not prevention.

---

## Overview

**Gains** — a mobile-first PWA for tracking weight training and cardio sessions. Built around a customizable split rotation system with automatic workout progression, PR tracking, session grading, and detailed history/stats.

**Live URL:** Deployed to Vercel via GitHub auto-deploy from `main` branch.
**Repo:** `github.com/bradenlanham/workout-tracker`
**Primary user:** Braden

---

## Tech Stack

- **Framework:** React 18.3.1 + React Router v6 (HashRouter)
- **State:** Zustand 4.5.2 with `persist` middleware (localStorage key: `workout-tracker-v1`)
- **Styling:** Tailwind CSS 3.4.3 with CSS custom properties for theming
- **Charts:** Recharts 2.12.2
- **Build:** Vite 5.2.0
- **Image export:** html2canvas (for share card JPEG export)
- **No backend.** All data lives in localStorage. Export/import via JSON backup files.

---

## File Structure

```
src/
├── App.jsx                    # HashRouter, route definitions, theme init, split auto-creation
├── main.jsx                   # React root mount
├── index.css                  # Tailwind config + CSS variables (obsidian/daylight themes)
├── theme.js                   # 10 accent color definitions (violet, blue, emerald, orange, rose, cyan, red, pink, white, black)
│
├── store/
│   └── useStore.js            # Zustand store — ALL app state + actions
│
├── data/
│   ├── exercises.js           # Built-in "BamBam's Blueprint" workout data (5 workouts, exercise groups)
│   ├── exerciseLibrary.js     # 140+ exercises by muscle group for the exercise picker
│   ├── splitTemplates.js      # Batch 17f — 6 curated templates for ChooseStartingPoint (BamBam / FullBody×3 / Upper-Lower×4 / PPL×3 / PPL×6 / Bro / 5×5) + loadTemplateForDraft(id)
│   └── hyroxStations.js       # Batch 37 — closed catalog of 8 HYROX stations (sta_skierg, sta_sled_push, sta_sled_pull, sta_burpee_broad, sta_row, sta_farmers, sta_sandbag_lunges, sta_wall_balls). Each carries id/name/dimensions/raceStandard. buildHyroxStationLibraryEntry(station) is the canonical converter consumed by buildBuiltInLibrary + migrateLibraryToV8.
│
├── utils/
│   └── helpers.js             # getNextBbWorkout, getLastBbSession, perSideLoad, getExercisePRs, isSetPR, isPR,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradenlanham/workout-tracker](https://github.com/bradenlanham/workout-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
