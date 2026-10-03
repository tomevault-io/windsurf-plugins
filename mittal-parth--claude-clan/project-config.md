---
trigger: always_on
description: A repository rendered as an isometric city, driven live by Claude Agent SDK
---

# Claude City — working notes

A repository rendered as an isometric city, driven live by Claude Agent SDK
sessions. `README.md` is the product and setup documentation; **this file is
how to work in the codebase.** Read it before touching the game layer.

---

## Commands

```bash
pnpm dev         # server + web against this repo
pnpm test        # vitest in every package that has tests
pnpm typecheck   # tsc --noEmit everywhere
pnpm build       # build all workspaces

pnpm --filter @sudo-city/web test          # one workspace
pnpm --filter @sudo-city/cli start -- ../other-repo
```

Node >= 22.5, pnpm 11.10. **Always run `pnpm typecheck` and `pnpm test` before
calling a change done** — there is no linter, so the compiler and the suite are
the only gates.

Web app: `http://127.0.0.1:5173`. Server WebSocket: `ws://127.0.0.1:4100/ws`.

---

## The pipeline

Everything downstream of a repo scan is a pure transform until it reaches
Phaser. Know where you are in this chain before changing anything:

```
repo ──worldgen──> WorldMap ──layout──> WorldSnapshot ──buildTerrain──> TerrainGrid ──WorldScene──> sprites
       (files,                (districts,              (per-tile kind,      (baked textures,
        imports,               plots, size)              road masks,          depth-sorted)
        churn)                                           props)
```

| Workspace | Responsibility |
| --- | --- |
| `packages/worldgen` | Scans a repo into a `WorldMap`. |
| `packages/layout` | `WorldMap` → `WorldSnapshot`: treemap districts, plot allocation. |
| `packages/protocol` | Shared zod schemas **and shared geometry constants**. |
| `packages/world` | SQLite persistence (`.sudocity/world.db` in the target repo). |
| `packages/agent` | Agent SDK sessions → `GameEvent`s. |
| `packages/cities` | PR worktrees and diff overlays. |
| `apps/server` | Fastify + WS: scan, snapshot per city, relay commands. |
| `apps/web` | Vite + React + Phaser client. |
| `apps/cli` | Boots server + web against a target repo. |

### Facts that bite

- **Plots are persisted.** `layoutWorld` takes `previousPlots` and keeps a
  file's plot across revisions so buildings don't shuffle. If you change what
  makes a plot invalid, handle the persisted-but-now-invalid case explicitly —
  otherwise stale worlds render wrong until someone regenerates them.
- **A PR city must be geometrically identical to `main`.** `squarify` reweights
  every rectangle when a single file's line count changes, so PR cities pass
  `main`'s own `snapshot.districts` and size. Never recompute districts for a PR.
- **Plots sit on odd lanes, streets on even ones.** `findPlotInDistrict` insets
  by 1 and steps 2; `BLOCK_STRIDE` is 6. That invariant is what guarantees a
  building is never planted on a street. Don't break it from either side.
- **Snapshots can be served from cache.** A change to layout rules doesn't
  retroactively fix worlds already generated. Renderer-side backstops are
  sometimes warranted (see the capitol reserve).

---

## The isometric world

This is where most of the work — and nearly all of the subtle breakage — lives.

### The one formula

`apps/web/src/game/iso.ts` places tiles; `createBaker().at()` in
`textures.ts` draws inside a texture. Same projection:

```
screenX = originX + (u - v) * HALF_W      // HALF_W = 48
screenY = originY + (u + v) * HALF_H - z  // HALF_H = 24
```

A `Point3` is `[u, v, z]` — **u and v are tile units on the ground plane, z is
pixels straight up.** Never mix them.

| You want | Do this |
| --- | --- |
| Move right on screen | `+u` and `−v` equally |
| Move up on screen | `−u` and `−v` equally |
| Move down-right | `+u` alone |
| Move down-left | `+v` alone |

Consequences worth memorising:

- **`u + v` is depth.** Larger is nearer the camera. It is the sort key for
  `setDepth`, for face order inside a bake, and for the order in which the
  masses of one prop are drawn.
- **`u − v` is the whole of screen x.** A quad whose corners share the same
  `u − v` has *zero width* and renders as nothing. Vertical elements with
  extent in both ground axes must be boxes.
- **`+u` and `+v` are 127° apart on screen, not 90°.** This is why sprites can
  never be rotated — see `isometric-animation`.

### Everything is baked, nothing is live

Props are drawn once into a Graphics, baked to a GPU texture, and instantiated
as sprites. A live Graphics per object is a draw call per object. **The unit of
work is a `bake*` function, not a render loop.**

Terrain goes further: every ground tile is a frame of one atlas
(`TERRAIN_ATLAS_KEY`), so the whole plane batches. Adding a `TerrainKind` means
adding a variant count and a bake, and it stays batched — that is cheap and
usually the right move.

> `WorldScene.ts` uses individual Sprites for ground tiles on purpose. A
> Blitter batches them into one object but silently stops drawing partway
> through a field this size, and a renderer that quietly loses terrain is worse
> than one that costs frames. Don't "optimise" it back.

### Ground belongs to terrain, not to a prop

The single most expensive lesson from building the capitol. Baking a landmark's
grounds — lawn, forecourt, the road ringing it — into its texture produces, all

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mittal-parth/claude-clan](https://github.com/mittal-parth/claude-clan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
