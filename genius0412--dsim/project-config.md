---
trigger: always_on
description: An FTC driver-practice sim hosting **more than one game**:
---


# CLAUDE.md — DSIM, an FTC driver-practice simulator

An FTC driver-practice sim hosting **more than one game**:

| id | game | status |
|----|------|--------|
| `decode` | **DECODE presented by RTX** (FTC 2025–26) | full match, scored, ranked |
| `chain` | **Chain Reaction** (2026 Unofficial-FTC CAD competition) | full match, scored, ranked |
| `biobuzz` | **BIOBUZZ presented by RTX** (FTC 2026–27) | full match, scored, ranked; 2D or 3D physics + view |

Vite + React + TypeScript, Canvas 2D. The CLIENT bundle is React + **Rapier 2D**
(`@dimforge/rapier2d-compat`, wasm) and nothing else; the rest of `dependencies`
(`ws`, `tsx`, `pg`, `jose`, `@neondatabase/auth`) exists for the SERVER and auth — keep the
client that lean. BIOBUZZ 3D adds two LAZY chunks (Rapier 3D deterministic, Three.js), fetched
only when a 3D world is stepped or drawn locally; `npm run bundleaudit` ratchets every chunk. Deploys to Vercel zero-config; a Node/`ws` authoritative game server on
Fly; Electron wrapper for the desktop build.

**The app brand is DSIM; a game is a "season" (`src/seasons.ts`).** Keep them separate in
UI copy — DECODE/Chain Reaction are what is currently loaded, not the product name.

> Read this file top-to-bottom once, then the guide for whatever you are about to touch —
> see the routing table below. When you touch a game, check whether the thing you're editing
> is shared (`src/sim/`, `src/config.ts`) or game-owned (`src/games/<id>/`) — that
> distinction is the single most load-bearing fact in the repo.

## ⚠️ READ THE AREA GUIDE FOR WHAT YOU ARE TOUCHING

This file is the CORE: what is true everywhere, and where everything else is. It used to be
2,200 lines (~43,000 tokens) and it is loaded into **every session**, so every session paid
for DECODE's gate-lever geometry whether it was going near a gate or not. The deep rules now
live in `docs/area/`, and **they bind exactly as hard as they did when they were in this
file.** Almost every paragraph in them is a bug that shipped, was measured, and was written
down so it would not ship twice.

**Before your first edit under a path below, read its guide.** Not skim — the rules there are
mostly of the form "the obvious thing is wrong, and here is the measurement that says so".

| you are touching | read first | ~tok |
|---|---|---|
| `src/sim/**` · `src/config.ts` · `src/math.ts` · `src/types.ts` | [docs/area/physics.md](docs/area/physics.md) | 7.2k |
| `server/**` · `src/net/**` · `src/game.ts` · replays | [docs/area/netcode.md](docs/area/netcode.md) | 6.8k |
| `server/db/**` · ranked · matchmaking · standing · admin | [docs/area/accounts.md](docs/area/accounts.md) | 7.0k |
| `src/ui/**` · `src/input/**` · `src/render/**` · `src/tutorial/**` · **strings** | [docs/area/ui.md](docs/area/ui.md) | 3.1k |
| DECODE rules — `src/sim/goal.ts`, `penalties.ts`, `field.ts`, `src/games/decode/**` | [docs/area/decode.md](docs/area/decode.md) | 10.6k |
| `src/games/chain/**` | [docs/area/chain.md](docs/area/chain.md) | 4.5k |
| `src/games/biobuzz/**` · `scripts/smoke-biobuzz/**` | [docs/area/biobuzz.md](docs/area/biobuzz.md) | 2.1k |
| adding a game — `src/games/types.ts`, `index.ts`, `sim.ts`, `src/seasons.ts` | [docs/area/adding-a-game.md](docs/area/adding-a-game.md) | 1.0k |
| `src/ads/**` · `server/kofi.ts` · `src/legalText.ts` · `src/storageKeys.ts` | [docs/area/monetization.md](docs/area/monetization.md) | 2.2k |
| `src/sponsor.ts` · `src/ui/Sponsor.tsx` · `electron/**` | [docs/area/sponsor.md](docs/area/sponsor.md) | 0.9k |

⚠️ **`src/sim/` is TWO guides.** It is the shared deterministic core (physics.md) *and* it is
where DECODE's own rules live (decode.md) — they predate the game seam and were deliberately
not relocated. Editing `src/sim/penalties.ts` or `src/sim/goal.ts` means both.

Also on demand, not in the routing table because they are not keyed to a path:
`docs/ui-standard.md` (the CSS rules `npm run uiaudit` enforces) · `docs/deploy.md` ·
`docs/capacity.md` · `docs/netcodeplan.md` · `docs/roadmap.md` (the eight roadmap items and
their state) · `docs/coordination-board.md` (**retired** — `.coord.retired.json` is in the repo
root, every `coord` command is a no-op, ignore it) · `docs/handoff-archive.md`.

**These are links, NOT `@imports`.** A CLAUDE.md `@path` import is loaded eagerly into every
session, which is the exact cost this split exists to remove. If you find yourself converting
them, you are undoing it.

**Keeping it honest.** `npm run docaudit` checks that every guide is routed, every route
resolves, every `governs:` glob still matches real files, no directory under `src/` or
`server/` has fallen through the gaps, and that this file stays under its token budget. That
last one is a RATCHET, the same way `uiaudit` is: the budget only ever goes down, so CLAUDE.md
cannot quietly grow back to 43k. If you add a rule here, ask first whether it belongs in a
guide — the test is whether a session working somewhere else needs to know it.

## Session protocol

**At the end of every working session, write/refresh `HANDOFF.md`** (repo root): current
state (is the build green?), what was finished, exact next steps, and gotchas. Read it at
session start if it exists — it may describe uncommitted mid-refactor state. HANDOFF is a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genius0412/dsim](https://github.com/genius0412/dsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
