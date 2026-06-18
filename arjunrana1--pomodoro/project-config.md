---
trigger: always_on
description: Orientation for Claude. The goal: jump directly to the right file without re-reading the codebase. If a section below covers your task, trust it and edit; only read other files when explicitly told "in doubt, verify."
---

# CLAUDE.md — Pomodoro Focus

Orientation for Claude. The goal: jump directly to the right file without re-reading the codebase. If a section below covers your task, trust it and edit; only read other files when explicitly told "in doubt, verify."

---

## Repo layout

```
/                          ← repo root (docs, PRD, CHANGELOG live here)
  PRD.md                   ← canonical product spec — source of truth for behavior + copy
  DESIGN.md                ← color palette, typography tokens, glassy aesthetic
  design_tokens.md         ← extended design system reference
  CHANGELOG.md             ← released-version log
  test-report.txt          ← manual QA evidence (53 test cases for v2)
  reference_designs/       ← PNG mockups, including v2/
  design_assets/           ← raw design source files
  test-evidence/           ← screenshot outputs from manual QA runs (gitignored beyond reports)
  app/                     ← the actual Vite + React app
```

All code lives under `app/`. Outside of `app/`, files are docs/assets only.

```
app/
  package.json             ← npm scripts (dev / build / lint / preview)
  vite.config.ts           ← Vite + Tailwind v4 plugin config
  index.html               ← SPA entry; mounts main.tsx
  src/
    main.tsx               ← React root, mounts <App />
    App.tsx                ← Top-level router by state.status (idle/running/paused/complete)
    App.css                ← Global one-off styles (glass utilities, animated bg)
    index.css              ← Tailwind directives + base resets
    store.ts               ← THE single source of truth for app state (useAppState hook)
    types.ts               ← All TypeScript types (AppState, PlanItem, NoteItem, DayRecord, FocusHistory)
    utils.ts               ← Pure helpers (formatTime, formatHours, date keys, history math)
    audio.ts               ← WebAudio oscillator-based sound effects
    assets/                ← Static image assets imported by components
    components/            ← All UI components (see map below)
```

---

## Component map — UI editing index

Each row tells you exactly which file owns what visual surface. **Edit only the file listed.** Files are short (most under 250 lines) — when you open one, you have the whole feature in context.

| Surface / Feature | File | Notes |
|---|---|---|
| Home screen layout, brand mark, preset pills, custom-duration pill, daily stats, marketing footer | `components/HomeScreen.tsx` | Footer copy is at the bottom (sections: What is, Pomodoro Technique, How to Use, Features). The marketing footer copy is canonical-spec'd in `PRD.md §7.7`. |
| Active session orb, timer display, Pause/Resume/Stop controls, locked left rail, plan-task checklist during session | `components/ActiveSession.tsx` | The orb's shimmer pauses via `animationPlayState`. Task circles are 20 px with 2.5 px primary border. |
| Flow Complete modal (post-session summary), session focus time, tasks accomplished, session notes, New Session CTA | `components/FlowComplete.tsx` | Shows `lastSessionElapsedSeconds` (stopped) or `initialSeconds` (natural). |
| Session Plan drawer (left side) — task title input, minutes input, Add Task button, drag-reorder list, Start Focused Session CTA | `components/SessionPlanSidebar.tsx` | Terminology must be "Task" / "Add Task" — never "Sub-task" (PRD §12). |
| Notes drawer (right side) — note input, edit/delete list, click-outside-to-close | `components/NotesDrawer.tsx` | Click-outside handler at lines ~53–66. |
| Focus History dashboard — 7-day bar chart, 2×2 stats grid, hourly heatmap, empty state | `components/FocusHistoryDashboard.tsx` | Reads `focusHistory.days`. Bar chart uses `formatHoursDecimal`. Heatmap is 24 hourly columns, 5 intensity tiers in `intensityClass()`. |
| App brand mark (logo SVG) | `components/BrandMark.tsx` | 35×35 SVG. Used in HomeScreen + ActiveSession headers. |

The 7-component list is complete — there is nothing else under `components/`.

---

## State, persistence, audio — non-UI but UI-adjacent

| What | File | Key exports / shapes |
|---|---|---|
| App state hook + all mutations | `store.ts` | `useAppState()` returns `{ state, selectPreset, commitCustomMinutes, startSession, pauseSession, stopSession, addNote, ... }`. Timer is wall-clock-driven (`computeRemainingFromWallClock`). |
| All TypeScript types | `types.ts` | `AppState`, `PlanItem`, `NoteItem`, `DayRecord`, `FocusHistory`, `SessionMode`, `AppStatus`. Add fields here first, then thread through `store.ts`. |
| Pure helpers | `utils.ts` | `formatTime`, `formatHours`, `formatHoursDecimal`, `getTodayKey`, `lastNDates`, `dayInitial`, `migrateDayRecord`, `recordFocusedSeconds`, `emptyDayRecord`. Hour-bucket math lives here. |
| Sound effects | `audio.ts` | `playClickSound`, `playStartSound`, `playPauseSound`, `playResumeSound`, `playStopSound`, `playCompletionSound`. All gated on `state.soundEnabled` in `store.ts`. |

### localStorage keys (current)

- `pomodoro-focus-state` — full app state minus history (set by `persistState`)
- `pomodoro-focus-stats` — `{ dateKey, focusSeconds, sessionsCount }` for today

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arjunrana1/pomodoro](https://github.com/arjunrana1/pomodoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
