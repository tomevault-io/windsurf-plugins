---
trigger: always_on
description: Guidance for AI coding agents working in this repository. The reader is assumed
---

# AGENTS.md

Guidance for AI coding agents working in this repository. The reader is assumed
to know nothing about the project.

## Project overview

`@wangjunjian/dsh-spritely` (**Spritely**) is a **DeepSeek Harness
(DSH) client plugin** that mounts a floating work-state mascot into the DSH
`web` UI. The sprite animates with the agent's live activity (idle, thinking,
writing, working, waiting, error, done), tracks the cursor with its gaze, can
be dragged anywhere (position persisted), offers switchable characters
(Blob/Bot/Cat/Ghost), and can recolor the app background (solid, gradient,
image URL, or local upload). The plugin also contributes a **selection
toolbar**: selecting text inside a conversation message floats an action bar
(copy, read aloud via SpeechSynthesis, quote-and-ask into the composer draft).

It is a **surface** plugin: the host entry (`src/index.ts`) exports an empty
`apply()` — there is no host-side behavior, no RPC channel, and no Config
schema. The package still ships a `cordis.patch.yml` declared via
`dsh.bundle.patch` so that `dsh plugin add` auto-inserts its loader row
(`id: ui-sprite`); the web half is picked up from the `dsh.client` metadata.

## Technology stack

- **Language**: TypeScript (strict), ESM (`"type": "module"`), React 18.
- **Runtime**: browser half only (`src/client/`), built on the DSH client slot
  system (`ctx.slots.inject` / `ctx.slots.register`) and the locale service.
- **Package manager**: pnpm 10 (pinned via `packageManager`; lockfile committed).
- **Build**: `tsc` (ES2024, bundler resolution) + `tsdown` (Rolldown-based
  bundler) + `lightningcss` for CSS Modules.
- **Lint/format**: Biome 2 (`biome.json`).
- **Tests**: Vitest 4 + jsdom + `@testing-library/react`.
- **Peer dependencies**: the `@deepseek-ai/dsh-*` packages are regular registry
  packages (release candidates); no local checkout of `deepseek-harness` is
  needed for development.

## Repository layout

```
src/
  index.ts                      # Host loader entry: empty apply() (no host behavior)
  invariant.ts                  # Cordis invariant companion (no-op installer, reserves ownership)
  css-modules.d.ts              # Ambient declarations for *.module.css imports
  client/
    index.ts                    # Client entry: locale dicts + shell.overlay registration
    sprite-state.ts             # Work-state source: derives SpriteActivity from the sessions service
    background-source.ts        # Persisted background source + BackgroundPresenter (CSS var applier)
    backgrounds.ts              # Background value shapes and presets
    sprite-kind-source.ts       # Persisted selected-character source (localStorage)
    sprite-position-source.ts   # Persisted dragged anchor position (localStorage)
    selection-toolbar.ts        # Selection tracker (snapshot store) + Markdown quote formatter
    sprites.tsx                 # SpriteKind registry: per-character SVG/pose metadata
    SpriteMascot.tsx            # The mascot component (menu, drag, gaze, poses)
    SpriteMascot.module.css     # CSS Modules next to their component
    SelectionToolbar.tsx        # Floating copy/read-aloud/ask bar over in-conversation selections
    SelectionToolbar.module.css
tests/
  setup.ts                      # window.__ModuleLoader__ polyfill + registry-backed require
  apply.client.spec.tsx         # Entry integration test: real Cordis Context + slot registry
  *.client.spec.ts(x)           # Source and component tests (jsdom via pragma)
images/                         # README screenshots (published to npm)
cordis.patch.yml                # Patch layer inserting the ui-sprite loader row
tsdown.config.ts                # Browser-bundle build (lazy-CJS factory for the web shell)
vitest.config.ts                # Test environments and setup
biome.json                      # Lint/format rules
.github/workflows/ci.yml        # CI: lint, typecheck, test, build
.github/workflows/release.yml   # npm publish on GitHub release (trusted publishing)
lib/                            # Build output (gitignored); do not edit by hand
```

## Build and test commands

```sh
pnpm install           # install dependencies (pnpm 10); also runs `prepare` (a build)
pnpm run typecheck     # tsc --noEmit
pnpm run build         # tsc -b && tsdown  (two-step, see below)
pnpm run test          # vitest run
pnpm run test:watch    # vitest
pnpm run lint          # biome check .
pnpm run lint:fix      # biome check --write .
pnpm run format        # biome format --write .
```

CI (`.github/workflows/ci.yml`) runs `lint`, `typecheck`, `test`, `build` in
that order on Node 22 with `pnpm install --frozen-lockfile`. Keep all four
green before considering a change done.

### Build pipeline (important)

The build is deliberately two-step:

1. `tsc -b` compiles `src/` to JS + declarations under `lib/types/`
   (`outDir: lib/types`). Source imports use `.ts` extensions, rewritten to
   `.js` on emit (`rewriteRelativeImportExtensions`).
2. `tsdown` (see `tsdown.config.ts`) rebundles `lib/types/client/index.js`
   into `lib/client.js`: a single CJS file wrapped in a
   `window.__ModuleLoader__.load({ id, factory })` banner/footer so the DSH web
   shell can load it. It also bundles the node-half entries (`lib/index.js`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wang-junjian/spritely](https://github.com/wang-junjian/spritely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
