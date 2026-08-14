---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal résumé website whose centerpiece is a fully in-browser 3D "detective office" built with **React Three Fiber / three.js**. The room is an interactive puzzle game: exploring and solving puzzles reveals the author's CV. There are effectively **two products in one repo**: the résumé site (`/[locale]/resume`, localized EN/NL) and the detective game (`/detective-room`, the bulk of the code).

The room is **100% procedural geometry** — there are no imported `.glb`/`.gltf` models. Every object is composed from primitives via a small custom "room engine". `README.md` is the developer-onboarding doc (how the engine works, tutorials for adding objects/puzzles); read it before doing engine work.

## Commands

```bash
npm run dev            # Next.js dev server (Turbopack) → http://localhost:3000
npm run build          # production build (NOTE: ESLint is skipped during build)
npm start              # run the production build

npm test               # Jest unit tests (jsdom). tests/Perf/ is excluded here.
npm test -- <pattern>  # single file, e.g. npm test -- state.data
npm test -- -t "name"  # single test by name
npx jest src/components/Game/__tests__/state.data.test.ts   # exact path

npm run lint           # `eslint .` (flat config). Clean: 0 errors; ~140 `no-explicit-any` warnings are expected signal, not failures.
npm run test:perf      # Playwright FPS/perf spec (needs the app running)
npm run ci:playwright  # start prod server + run perf spec
npm run lhci           # Lighthouse CI
```

CI (`.github/workflows/ci-cd.yml`, on push/PR to `master`): `npm ci` → `npm test -- --runInBand` → `npm run build` → Playwright perf → Lighthouse. **There is no lint step in CI**, and lint is not enforced anywhere.

## Architecture — the big picture

**Routing / entry.** `src/app/detective-room/page.tsx` → `DetectiveRoomClient.tsx` (tutorial overlay, boot/preload gate, all HUD providers) → dynamically imports `DetectiveRoom.tsx` with **`ssr: false`** (the whole game is client-only; anything touching `window`/WebGL must stay out of SSR). `DetectiveRoom.tsx` owns the `<Canvas>` and wires the scene, camera/input, post-processing, and the inspect overlay. The résumé site lives under `src/app/[locale]/` and is a separate, simpler surface.

**Model-building stack (read these together).** `@/*` maps to `src/*` (tsconfig).
- `Models/Generic/Outlined/Outlined.tsx` — one mesh + a scaled back-face outline mesh + hover/click/inspect + optional texture and magnifier-reveal material.
- `Models/Generic/Outlined/FramedPlane.tsx` — framed poster/screen primitive (used for puzzle "clue" images), with a magnifier-only visibility mode.
- `Models/Generic/ModelGroup.tsx` — the central abstraction: takes an array of `PartSpec`, builds many `Outlined` parts, resolves per-part vs group vs `materialsById` overrides, manages auto/manual interaction hitboxes, and emits the inspect payload. Every concrete model (Bookshelf, Desk, Clock, …) is a thin wrapper that computes `parts` and hands them to `ModelGroup`.

**Scene composition.** `DetectiveRoom.tsx` renders **cluster** components (`DetectiveRoom/Clusters/*`: Walls, BigFurniture, BindersAndBooks, Lights, AnimatedDecoration, FlatDecoration, Decoration) plus **functional-object** groups (`FunctionalObjects/*`: MovingObjects, PuzzleObjects, UsableItemObjects, Effects). Add new objects to the appropriate cluster, never directly to the scene root.

**Anchors are the single source of truth for placement.** `Game/anchors.ts` holds position/rotation (and optional `eye` for camera focus) for every object and camera target. Clusters read `ANCHOR.*`; right-click focus and post-solve camera zooms are computed from anchor `eye`/`position`.

**Game state is a hand-rolled external store — NOT Redux/Zustand/Context.**
- `Game/state.logic.ts` — a `GameState` class with an immutable `GameSnapshot` (files, drawer_files, poofs, drawers, puzzlesConfig, puzzleStatus, cardboardBoxes). Every mutation creates a new snapshot and calls `emit()`.
- `Game/state.ts` — a **module-level singleton** `gameState`, plus `useGameState()` (subscribe + force re-render) and `useGameActions()` (returns the singleton). The singleton persists across mounts/HMR and has no reset method; tests import the class directly.
- `Game/state.data.ts` — `initialSnapshot` + all puzzle definitions. **This is where the résumé content lives** (see below). Branded IDs via `asPuzzleId`/`asFileId`; `state.data.test.ts` enforces cross-references between `puzzlesConfig`/`puzzleStatus`/anchors stay in sync.

**Puzzle / inspect flow.** Clicking an object calls `openInspect(state)`; `Puzzles/ObjectInspectOverlay.tsx` renders it **in its own separate `<Canvas>`** (on-demand `invalidate()` rendering, unlike the always-rendering main scene). Solving reports back up through `DetectiveRoom`'s `onSolved`/`onAction` callbacks, which drive game-state actions (`pinPuzzle`, `handleSecretOpen`, `requestOpenCardboardBox`) and a choreographed set of timers for view-delay → camera zoom. Solved puzzles get pinned to the corkboard (`Puzzles/PuzzleNode.tsx`, `RedStringsEffect.tsx`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Novereem/detective-resume](https://github.com/Novereem/detective-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
