---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Thinking Orbs** (thinkingorbs.com) — animated dotted-sphere status indicators for AI agents: one sphere, a state for each thing an agent does (working, reasoning, compacting, searching, background tasks, retrying). Distributed two ways from one source: the npm package (`npm i @yogesharc/thinking-orbs`, `import { Orb } from "@yogesharc/thinking-orbs"`, or `mountOrb` from `@yogesharc/thinking-orbs/vanilla` without React), and a shadcn registry item (`npx shadcn@latest add https://thinkingorbs.com/r/orb.json`) for people who want to own and edit the source. There is no CLI of our own.

The npm package is `@yogesharc/thinking-orbs`. npm refused the unscoped `thinkingorbs` as too similar to the existing `thinking-orbs`, an unrelated package with the same pitch, so keep the scope: dropping it installs theirs.

Only the homepage ships to users. `/playground` (the orb playground) and the `/v1`–`/v4`, `/v6` layout trials are committed but unlinked and noindexed. The old design playground (`app/playground-old/`, `prototype/`, and the `motion` and `status` pages) stays gitignored — never commit those paths.

## Commands

Run from the repo root:

```bash
pnpm dev      # Next.js dev server (Turbopack) on :3000
pnpm build    # production build
pnpm lint     # eslint
```

No test runner is set up yet. Don't invent one without asking. Type-check with `npx tsc --noEmit -p .` from `apps/web` (TypeScript isn't installed at the root).

**Install quirk:** `create-next-app` wrote an `apps/web/pnpm-workspace.yaml`, which makes `apps/web` its own pnpm workspace root — so the lockfile and `node_modules` live in `apps/web/`, not at the repo root. The root scripts work anyway (they shell out via `--filter web`). `packages/thinkingorbs` has its own `pnpm-workspace.yaml` the same way, so run `pnpm install` / `pnpm build` inside it.

## Architecture

```
apps/web/            Next.js 16 site — also serves the registry
  components/        site UI: the chat mock and its styles
  registry/orb/      what ships: orb-core.ts (all the drawing, plain JS, `mountOrb`), orb.tsx (the React <Orb> around it), and the opt-in shapes.ts and renders.ts
  app/               the homepage; app/llms.ts holds the docs as markdown; app/r/[item] (orb, orb-shapes, orb-renders .json) and app/llms.txt serve them
packages/thinkingorbs/  the npm package; `tsc` compiles both registry/orb files into dist/
```

### The orb

`registry/orb/orb-core.ts` draws every orb: one SVG of imperatively created circles, redrawn each frame. `VARIANTS` lists every public `state` and the `variant`s it comes in, `default` first (`<Orb state="working" variant="gyro" />`); variants are named after their motion. Internally the drawing keys on a `Look`: the state alone for `default`, else `state-variant`. Where the dots sit comes from `distribute()` (a Fibonacci sphere, or spiral arms for `background-spiral`); `density`, `dotSize` and `tilt` tune it; every behaviour on top of the spin is a branch keyed on the state inside the one `draw` loop, and spin periods come from `PERIOD`. Renaming a state or variant means changing it in `orb-core.ts`, `app/orbs.tsx`, `app/llms.ts` and `packages/thinkingorbs/README.md`.

The core ships only the sphere drawn in dots. Other shapes and renders are opt-in plug-ins, so they cost nothing unless imported: `registry/orb/shapes.ts` (`OrbShape`: `points(count, look)` plus optional `scale` and `tip`) and `registry/orb/renders.ts` (`OrbRender`: `mount` returns a per-point `dot()` and a per-frame `frame()`; `flat` renders like halftone and lines drop the tilt, and Working's ring runs straight down). They ship as `@yogesharc/thinking-orbs/shapes` and `@yogesharc/thinking-orbs/renders`, and as the `orb-shapes` and `orb-renders` registry items. Not every state is tuned for every extra, by design.

### The homepage

`app/page.tsx` renders `Orbs` from `app/orbs.tsx`. A row along the top holds the contents (Orbs · Installation · Usage) on the left and Sponsor, GitHub stars and the theme toggle on the right, pinned on wide screens with no fill. At `xl` it's three columns: a sticky left column with just the pitch (install bar with Copy prompt, credit and MIT License), the orb cards two to a row in story order, and a sticky `components/chat-mock.tsx`; Installation and Usage follow below the cards, ending in an llms.txt link. There's no footer. Whichever card crosses a line 35% down the viewport is "picked" — a row splits its height between its cards, so scrolling zigzags left then right (clicking scrolls a card onto its share of the line) — the chat's live line shows it, and the rows above it build up as you scroll. Below `xl` the cards are a grid, the chat stays at its seed rows and tapping a card picks it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yogesharc/thinking-orbs](https://github.com/yogesharc/thinking-orbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
