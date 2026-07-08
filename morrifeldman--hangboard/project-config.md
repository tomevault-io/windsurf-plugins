---
trigger: always_on
description: This file is the canonical onboarding doc for AI coding agents. Repo dir is still `~/hangboard/` for historical reasons; the user-facing app is **Cairn**. IndexedDB and localStorage keys also still use the `hangboard-*` prefix — never rename them.
---

# Cairn (Hangboard PWA) — Agent Onboarding

This file is the canonical onboarding doc for AI coding agents. Repo dir is still `~/hangboard/` for historical reasons; the user-facing app is **Cairn**. IndexedDB and localStorage keys also still use the `hangboard-*` prefix — never rename them.

## Stack
Vite 7 · React 19 · TypeScript 5.9 (strict) · Tailwind 3 · Zustand 5 · vite-plugin-pwa 1.3 · Recharts · Vitest 4 · Playwright 1.58

## Commands

```bash
npm run dev          # dev server on :5173 (normal timers: 7s hang, 3s rest, 180s break)
npm run build        # tsc -b && vite build → dist/
npm run test:unit    # Vitest (~155 cases, no browser)
npm run test:unit:watch
npx playwright test  # E2E suite (auto-starts dev server with VITE_TEST_MODE=true)
```

A pre-commit hook at `scripts/pre-commit` runs `npm run build`. Install once: `cp scripts/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit`.

## Version Control: use `jj`, not raw `git`

The repo is a colocated jj+git checkout (`.jj/` alongside `.git/`). Both work, but the user's surface is jj. Common equivalents:

| Want | Use |
|------|-----|
| status / diff | `jj st`, `jj diff` |
| commit | `jj describe -m "msg"` (sets message on @, the working-copy change) |
| start new change | `jj new` |
| move main forward | `jj bookmark set main -r @-` |
| push | `jj git push --bookmark main` |

Branches are **bookmarks** in jj and don't auto-advance. Typical flow:

```
# make edits
jj describe -m "Commit message"
jj new
jj bookmark set main -r @-
jj git push --bookmark main
```

## Architecture

### Core
- `src/data/holds.ts` — `HoldDefinition` interface + `HOLDS` array (Workout A). No Vite env imports, safe to use in tests/Node.
- `src/data/workout-b.ts` — `HOLDS_B` (Workout B: MacLeod Max Hang, 9 items, 3 sets, 5lb increments).
- `src/data/workout.ts` — re-exports `HOLDS`, `HOLDS_B` + test-aware timer constants (`HANG_SECS`, `REST_SECS`, `BREAK_SECS`, `SET1_REPS`, `SET2_REPS`). Imports `import.meta.env` — do **not** import this from tests.
- `src/lib/stateMachine.ts` — pure `advancePhase`/`skipSet`/`skipNextSet`/`skipNextHold`. Per-hold `numSets`, `repsPerSet`, `isRestOnly`, `prepBetweenReps` flags.
- `src/lib/workoutTime.ts` — pure `setPhaseSequence` + elapsed/remaining math; tested in `workoutTime.test.ts`.
- `src/store/useWorkoutStore.ts` — Zustand store. Persists `weights`, `weightsB`, `weightsTest`, `selectedWorkout`, `gymDefaults`. Session phase/index state is **not** persisted (resets on page load).
- `src/hooks/useTimer.ts` — setInterval-based; `end = Date.now() + remaining * 1000`.
- `src/lib/audio.ts` — Web Audio API singleton, lazy init. `initAudio()` must be called inside a user gesture (Start button) to unlock the AudioContext on Android.
- `src/lib/haptics.ts` — `navigator.vibrate` is optional-chained → no-op on desktop.

### Persistence (IDB)
- DB: `"hangboard-history"`, version 5.
- Stores: `"sessions"` (workout records, including gym entries), `"climbs"` (route logs), `"notes"` (dated notes), `"schedules"` (planner days, `by-date` unique index), `"meta"` (device-local key/value, out-of-line keys — reminder config etc., **shared with the service worker**).
- Session index: `"by-start"` on `startedAt`.
- `getMeta`/`setMeta` (in `history.ts`) wrap the `meta` store. The SW reads it via the raw IndexedDB API (no `idb` import) since it can't share app modules that pull DOM globals.

### Workout types
- **Repeaters** (`workoutType: "repeaters"`) — Workout A, 2 sets of 7/6 reps, 7s hang / 3s rest.
- **Max Hang** (`workoutType: "max-hang"`) — Workout B, 3 sets × 1 rep, 10s hang, 5lb progressions.
- **Beginner** (`workoutType: "beginner"`) — single-set legacy.
- **Test** (`workoutType: "test"`) — short-timers state under `?test`; never saved.
- **Gym workout types** (`GymWorkoutType` in `src/lib/history.ts`): `arc | cir | pe-route | lbc | wbl | performance | hard-bouldering | limit-bouldering | injury | stretching`. Defined in `src/data/gymWorkouts.ts` with `fieldDefs` per type. `FieldType` supports `number | text | grade-v | grade-yds | select | multi-select`.

### Hold IDs (storage keys — never rename)
`jug, large-edge, mr-shallow, small-edge, imr-shallow, wide-pinch, sloper, med-pinch`

Max-hang hold IDs are separately prefixed (`b-chisel`, `b-hc`, `b-open`, etc.) and share no relation to the repeaters board.

### Progress screen
- `src/lib/progressData.ts` — `buildTrend`, `buildCalendar`, `calendarMonthLabels`, `computeStats` (pure).
- `src/lib/gradeTrends.ts` — `buildGradeTrend(climbs, "months"|"seasons"|"years")` for outdoor-sport route grade trend chart. Filters to `setting === "outdoor" && type === "sport"`, excludes attempts, returns SPORT_GRADES indices per style per bucket with empty buckets filled with nulls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morrifeldman/hangboard](https://github.com/morrifeldman/hangboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
