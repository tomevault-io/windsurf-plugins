---
trigger: always_on
description: > Pointer index for agents working in this repo. Keep this file lean: load linked
---

# claude-of-tanks - Agent Index

> Pointer index for agents working in this repo. Keep this file lean: load linked
> files on demand, prune no-op instructions, and keep generated facts inside
> `agent-docs:auto` blocks.

## Overview
<!-- agent-docs:fill:overview -->
Browser-native Three.js armored combat game. The runtime combines a fixed-step
60 Hz simulation, procedural first-party vehicle fleet, thirty battlefields,
garage/showroom presentation, bots, armor/ballistics/modules, and mobile input.
Treat current `origin/main` as active shared work: isolate broad changes in a
worktree and never stage generated tank work wholesale.

## Architecture Pointers
<!-- agent-docs:fill:architecture -->
- `docs/DEVELOPMENT.md#publishing-to-shared-main` — shared Codex/Claude integration,
  overlap review and preflight; read before integrating or publishing changes.
- `docs/ARCHITECTURE.md` — original module contracts and simulation invariants.
- `docs/MULTIPLAYER-ARCHITECTURE.md` — authoritative multiplayer migration.
- `docs/tank-generation/README.md` — source/markup intake, measured construction,
  prompts, quality gates and resumable tank-generation handoffs. Read before
  new source-backed tanks, family rebuilds or exact-surface geometry repairs.
  Begin with [`docs/tank-generation/SKILL.md`](docs/tank-generation/SKILL.md);
  this documentation-only directory is not discovered by the source-code index.
- `docs/tank-generation/fleet-style-performance-priority.md` — urgent open
  fleet-wide primitive/cost, switching, roller/track, chassis-closure and
  material-role work; read before adding further tank micro-detail.
- `src/main.ts` — strict typed boot, scene composition, UI flow, and render-loop
  wiring across extracted lifecycle owners; keep changes surgical.
- `src/game/stateCore.ts` — dependency-free typed session shell, event bus,
  and deterministic integration RNG.
- `src/game/state.ts` — battle roster and authoritative simulation integration.
- `src/sim/` — movement, armor, damage, spotting, and ballistics logic.
- `src/net/` — transport-independent protocol, lobby, authority, and snapshots.

## Stack
<!-- agent-docs:auto:stack start -->
- **Name:** claude-of-tanks
- **Package manager:** npm
- **Languages:** typescript
- **Framework:** vite
<!-- agent-docs:auto:stack end -->

## Commands
<!-- agent-docs:auto:commands start -->
- Package scripts detected: 122. Use `package.json` as the exhaustive source.
- `npm run build` - VITE_PUBLIC_BUILD=1 vite build && node tools/strip-nc-assets.mjs
- `npm run test` - node tools/run-selftests.mjs core
- `npm test` runs the pre → core → post receipt groups; a receipt whose observable inputs are
  byte-identical to its last PASS is skipped with a SKIP line (`COT_SELFTEST_CACHE=0 npm test`
  runs everything), every failure in a group is reported in one run (`COT_SELFTEST_FAIL_FAST=1`
  stops at the first), and `tank:release:check` runs its fleet probes beside the suite and the
  build after the serial scoring steps. See docs/DEVELOPMENT.md "Fast checks".
- `npm run typecheck` - node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit && node tools/core-unused-check.mjs
- `npm run agent-docs` - node scripts/run-agent-docs.ts
- Keep this block compact. Put full command catalogs in a generated command index, not in AGENTS.md.
<!-- agent-docs:auto:commands end -->

## Directory index
<!-- agent-docs:auto:dirmap start -->
| Directory | Skill | Purpose |
|---|---|---|
| `api/` | [`api/SKILL.md`](api/SKILL.md) | Maintain deployed signaling, ICE credential, and public GitHub-count HTTP entrypoints. |
| `server/` | [`server/SKILL.md`](server/SKILL.md) | Implement and operate Claude of Tanks signaling and dedicated authoritative multiplayer servers. |
| `src/` | [`src/SKILL.md`](src/SKILL.md) | Navigate browser boot and shared source contracts while preserving subsystem and bundle boundaries. |
| `src/app/` | [`src/app/SKILL.md`](src/app/SKILL.md) | Maintain typed application composition, lazy owner access, frame wiring, and combat warm lifecycle. |
| `src/audio/` | [`src/audio/SKILL.md`](src/audio/SKILL.md) | Work on event-driven spatial audio, radio voices, engines, weapons, ambience, and mix state. |
| `src/dev/` | [`src/dev/SKILL.md`](src/dev/SKILL.md) | Maintain opt-in diagnostics, bounded traces, deterministic shot controls, and capture readiness gates. |
| `src/docs/` | [`src/docs/SKILL.md`](src/docs/SKILL.md) | Maintain the public field manual, indexed topic pages, typed icons, and battle-reel interactions. |
| `src/engine/` | [`src/engine/SKILL.md`](src/engine/SKILL.md) | Work on renderer, lighting, camera, postprocessing, device quality, and frame diagnostics. |
| `src/fx/` | [`src/fx/SKILL.md`](src/fx/SKILL.md) | Work on pooled particles, impacts, destruction effects, decals, and shared FX time. |
| `src/gallery/` | [`src/gallery/SKILL.md`](src/gallery/SKILL.md) | Build and verify the public Tank Gallery, its technical dossiers, diagnostic overlays, and exact-surface markup review packets. |
| `src/game/` | [`src/game/SKILL.md`](src/game/SKILL.md) | Work on battle integration, bots, input, garage dressing, progression, replays, and studio state. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevin-Liu-01/Claude-of-Tanks](https://github.com/Kevin-Liu-01/Claude-of-Tanks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
