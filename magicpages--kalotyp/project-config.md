---
trigger: always_on
description: Guidance for AI coding agents (and humans skimming for the rules) working in
---

# AGENTS.md

Guidance for AI coding agents (and humans skimming for the rules) working in
the Kalotyp repository. This is the canonical instruction file; `CLAUDE.md`
points here.

Kalotyp is an open-source, MIT-licensed image editor for Ghost CMS, published as
`@magicpages/kalotyp-core` / `@magicpages/kalotyp-ui` / `@magicpages/kalotyp` (the
Ghost bundle).
Self-hosted Ghost users upload `kalotyp.js` and `kalotyp.css` through
**Settings → Integrations → Pintura**, toggle the integration on, and get image
editing in the post editor with **zero changes** to Ghost itself. ("Pintura" is
the name of Ghost's built-in image-editor integration slot — see Legal below.)

This is a working codebase. The notes below describe what's actually here.

## Setup & build

- **pnpm** workspaces. Install with `pnpm install`.
- `pnpm dev` — Vite dev playground (`apps/playground`), no Ghost needed.
- `pnpm build` — library build of the three published packages via Vite.
- `pnpm test` — Vitest unit tests across core / ui / ghost.
- `pnpm typecheck` — `tsc --noEmit` per package.
- `pnpm lint` — Biome check (lint + format). `pnpm lint:fix` to apply.
- `pnpm size` — bundle-size budget check.
- `pnpm test:e2e` — Playwright against a real containerised Ghost
  (`apps/ghost-test`; needs Docker + admin creds).

## Tech stack

- **TypeScript, strict mode.** No `any`, no unexplained `as` casts.
- **Vanilla DOM + plain CSS** for the UI. No React/Vue/Angular or framework
  runtime in the bundle — the single most important decision for the
  bundle-size budget.
- **Canvas 2D + OffscreenCanvas where supported.** No WebGL.
- **No Fabric.js, Konva, or Pixi.** The small subset we need is hand-written.
- **Vite** build (library mode), **Vitest** unit tests, **Playwright** E2E,
  **Biome** lint/format, **Changesets** versioning (the three published
  packages are linked at one shared version).

## Architecture

```
kalotyp/
├── packages/
│   ├── core/                 # framework-agnostic editor engine
│   │   └── src/
│   │       ├── canvas/       # image loading, viewport, bake
│   │       ├── events/       # event bus
│   │       ├── geometry/     # rect/point/size math
│   │       ├── history/      # snapshot-based undo/redo
│   │       ├── output/       # output state (mime + quality + strip-metadata)
│   │       ├── pipeline/     # chain runner, encoder, EXIF copier
│   │       ├── plugins/      # per-tool state + bake (no DOM here)
│   │       │   ├── crop/ rotate/ flip/ resize/ finetune/
│   │       │   ├── filter/   # shares finetune state via presets
│   │       │   ├── annotate/ redact/ frame/
│   │       └── state/        # observable store primitive
│   ├── ui/                   # default UI — DOM + plain CSS, no framework
│   │   └── src/
│   │       ├── canvas/       # render helpers for the stage
│   │       ├── cheatsheet/   # `?` keyboard cheatsheet modal
│   │       ├── dom/          # shell, focus trap, nested-modal helper
│   │       ├── output/       # save-caret popover
│   │       ├── preferences/  # per-site preferences modal + storage
│   │       ├── plugins/      # per-tool mount + panel DOM + CSS
│   │       └── styles/       # base / mobile / per-plugin stylesheets
│   └── ghost/                # Ghost adapter — produces dist/kalotyp.{js,css}
│       └── src/
│           ├── editor.ts     # the session — destructive-edit model
│           ├── install-global.ts  # sets window.pintura
│           ├── contract.ts   # public-facing types
│           ├── default-presets.ts
│           └── source-image.ts
├── apps/
│   ├── playground/           # Vite dev harness — no Ghost needed
│   └── ghost-test/           # docker-compose Ghost + Playwright E2E
└── docs/
    └── ghost-contract.md     # the spec, cited line-for-line from Ghost
```

### The session model: destructive-edit on tab switch

When the user leaves a tab with dirty plugin state, `commitActiveIntoChain` runs
that plugin's `bake` against the current working image, appends `(id, state)` to
a chain, and resets the plugin's store. The next tab mounts on the now-baked
working image. The save chain is the order the user actually used the tools —
there is no fixed chain order.

Implications: each plugin's state shape is what gets baked, not what later tools
read. Filter is a UI tab that shares the finetune slot's store, and its commit
path is remapped to bake through the finetune slot (its own bake is identity).
Undo restores both the chain and per-plugin state in one step; snapshots carry
the chain under a private `'__committedChain__'` key (see `captureSnapshot` /
`applyHistoryResult` in `editor.ts`).

### The split between `core`, `ui`, and `ghost`

Deliberate. The Ghost bundle composes the other two; a future host could ship a
different UI without touching the engine.

## The Ghost integration contract

`docs/ghost-contract.md` is the canonical reference, cited line-for-line to
`../core/ghost/admin/` (a `TryGhost/Ghost` checkout). Read it before changing
anything in `packages/ghost/`. **When uncertain about the contract, grep
`../core/ghost/`, not your memory.**

In one paragraph: Ghost loads the editor as an ES module via dynamic `import()`
from a URL stored in `settings.pinturaJsUrl`. The module assigns itself to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magicpages/kalotyp](https://github.com/magicpages/kalotyp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
