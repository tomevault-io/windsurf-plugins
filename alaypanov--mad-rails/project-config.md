---
trigger: always_on
description: Rail Builder — a 2D top-view, touch-first rail-laying browser game. Core mechanic: rails are pre-placed on the board as **sliding tiles**; drag one and it slides **axis-locked along its row/column** through as many empty cells as the drag allows, stopping at the first barrier (2D, 15-puzzle style) to assemble a connected path between the start and end stations. The board is **packed**: off-corridor land is filled with movable **blank filler tiles**, leaving only ~2–3 empty holes as maneuvering r
---

## Project

Rail Builder — a 2D top-view, touch-first rail-laying browser game. Core mechanic: rails are pre-placed on the board as **sliding tiles**; drag one and it slides **axis-locked along its row/column** through as many empty cells as the drag allows, stopping at the first barrier (2D, 15-puzzle style) to assemble a connected path between the start and end stations. The board is **packed**: off-corridor land is filled with movable **blank filler tiles**, leaving only ~2–3 empty holes as maneuvering room — so assembling the path means shuffling both rail tiles and blanks around the immovable anchors. Immovable anchors (tunnels through terrain) and denser terrain form maze-like corridors that constrain movement; tiles can't pass each other, so reaching each rail's slot requires shuffling. TypeScript + HTML5 Canvas + Vite. No external rendering/physics libraries; all drawing is procedural on a 2D canvas context.

## Commands

```bash
npm run dev       # Vite dev server (http://localhost:5171, see vite.config.ts), HMR
npm run build     # tsc -b (typecheck) then vite build → dist/
npm run preview   # serve the production build
npx tsc --noEmit  # typecheck only
```

### Logic smoke test

There is no test runner. The pure-logic self-test lives at `test/smoke.ts` and is run by bundling with esbuild (shipped with Vite) and executing under node:

```bash
npx esbuild test/smoke.ts --bundle --platform=node --format=esm --outfile=test/smoke.mjs && node test/smoke.mjs
```

It generates levels with scramble disabled and asserts the train reaches the end on the solved board (200 levels) and that an empty board never reaches the end; it also unit-tests `canStep`/`stepRail` edge cases (barriers, immovables, occupied neighbors, reversibility), the multi-cell `slideDistance`/`slideRail` primitives (barrier stop, atomicity, reversibility), and `Blank` tiles (no ports, movable, act as barriers, slide reversibly), plus a generation check that the board is packed (blanks placed, ~2–3 holes left) and the `Empty` region is 2-connected. It proves scrambled levels are solvable by **asserting Wilson's theorem preconditions** (2-connected `Empty` region, not a cycle, ≥2 identical blanks, ≥1 hole, rail multiset preserved by the swap) on a sample of scrambled levels — the preconditions ARE the proof, since a complete solver is infeasible at these sizes (it can never prove a board unsolvable). The `test/smoke.mjs` bundle is generated artefact — delete it after running.

## Architecture

The code is split into **framework-agnostic game logic** (`src/game/`, no DOM/canvas) and **presentation/glue** (`src/render/`, `src/input.ts`, `src/layout.ts`, `src/loop.ts`, `src/main.ts`). Keep that boundary: all gameplay rules live in `src/game/` and should be unit-testable; `src/render/` only reads game state and draws.

### Rail connection model (the core)

Each rail piece exposes a set of **ports** (subset of N/E/S/W). Two adjacent cells connect iff both expose a port facing each other (mutual). `src/game/pieces.ts` defines port sets per `(PieceType, orientation)`; `Straight`/`Tunnel` have 2 orientations, `Curve` has 4, `Blank` has 1 (an **empty** port set — no ports). `Straight` (movable) sits on `Empty`; `Tunnel` (immovable, straight-only) sits on `Terrain`; `Curve` (movable) sits on `Empty` only; `Blank` (movable, no ports) sits on `Empty` — a filler tile that is not track (entering one derails the train in `resolvePath`).

`src/game/slide.ts` is the canonical barrier logic for the movement: a cell is *passable* iff in-bounds, `Empty`, and no rail. `canStep(level,r,c,dir)` returns whether the movable tile at `(r,c)` can step **one cell** in direction `dir` (N/E/S/W) — the target must be in-bounds passable (empty land, no rail), so tiles are blocked by terrain/other tiles (rails or blanks)/stations/border and can't pass or swap with each other. The logic is piece-agnostic: any non-immovable `cell.rail` (Straight/Curve/**Blank**) moves; a Blank is a movable barrier to others. `stepRail(level,r,c,dir)` performs the one-cell step (relocate + free source). `slideDistance(level,r,c,dir)` counts how far the tile can slide straight in `dir` before hitting a barrier, and `slideRail(level,r,c,dir,steps)` performs an **axis-locked multi-cell slide** of `steps` cells — atomic (no mutation unless the full slide fits within `slideDistance`), and reversible. `isImmovable` (Tunnel) tiles never move and act as barriers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alaypanov/mad-rails](https://github.com/alaypanov/mad-rails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
