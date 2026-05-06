---
trigger: always_on
description: This file is the maintenance contract for this repo. Keep it aligned with the code.
---

# motion-solid (agent guide)

This file is the maintenance contract for this repo. Keep it aligned with the code.

## What This Repo Is

- `motion-solid` is a SolidJS-first Motion library built on top of upstream `motion-dom`.
- The local package version is `0.5.0`.
- The workspace currently pins published `motion-dom` `^12.35.1`.
- The component runtime is not a local store-driven animation engine. Motion component state lives on upstream `motion-dom` objects.
- The framework layer is a Solid translation of the framework-owned pieces that live in `motion/react`.
- This repo is not a byte-for-byte port of `motion/react`. Most remaining parity bugs live in the translated orchestration layer, not the `motion-dom` engine.

## Hard Rules

- Do not reintroduce a `createStore` / `MotionState`-style runtime for motion components.
- Treat `motion-dom` as the source of truth for component animation state, projection state, and feature execution.
- Treat `motion/react` as the semantic reference for framework behavior: presence, layout timing, feature mounting, and shared-layout orchestration.
- Keep the public API Solid-native:
  - Accessors where reactive values are exposed
  - normal Solid `ref` prop behavior
  - SSR/hydration-safe rendering
  - kebab-case-first style and transform keys
- Every change must update `AGENTS.md` in the same branch.
- If public behavior, API, defaults, timing, caveats, or divergences change, update all relevant docs in the same change:
  - `AGENTS.md`
  - `package/README.md`
  - docs pages under `docs/src/routes/docs/*.mdx`
- If a behavior intentionally differs from Motion semantics, add an explicit divergence note in docs and tests.

## Repository Map

- Monorepo root uses Bun workspaces:
  - `package` = library
  - `docs` = docs site
- Library source: `package/src`
- Library tests: `package/tests`
- Docs pages: `docs/src/routes/docs`
- Interactive demos: `docs/src/components/demos`
- CI workflows: `.github/workflows`
- Upstream `motion-dom` repo: `tmp/motion-upstream`

Docs app note:

- Client bootstrap lives in `docs/src/entry-client.tsx`.
- The current docs build emits a Vinxi warning about a missing default export from `src/entry-client.tsx`, but the build still succeeds. If touching docs bootstrap, verify the real built site behavior, not just the warning text.
- Current sidebar docs targets live under `docs/src/routes/docs`:
  - `getting-started.mdx`
  - `demos.mdx`
  - `architecture-caveats.mdx`
  - `motion-component.mdx`
  - `layout-animations.mdx`
  - `variants.mdx`
  - `animate-presence.mdx`
  - `motion-config.mdx`
  - `hooks.mdx`
  - `gestures.mdx`
  - `drag.mdx`
- The docs layout TOC reads `h2` and `h3` inside `.docs-content`. Demo components must not render heading tags for demo titles or internal labels, otherwise they pollute the page TOC.

Package publishing note:

- `package/package.json` explicitly includes `README.md` in the published `files` list so installed package contents still carry the local package readme even if pack tooling or workspace behavior changes.

## Commands

Package manager:

- `bun`

Root:

- `bun install --frozen-lockfile`
- `bun dev`
- `bun run lint`
- `bun run lint:fix`
- `bun run format`
- `bun run format:check`

Library:

- `bun --filter motion-solid build`
- `bun --filter motion-solid typecheck`
- `bun --filter motion-solid test`
- `bun --filter motion-solid test:watch`
- `bun --filter motion-solid test:browser`

Docs:

- `bun --filter @motion-solid/docs dev`
- `bun --filter @motion-solid/docs build`
- `bun --filter @motion-solid/docs start`
- `bun --filter @motion-solid/docs typecheck`

## Architecture

### 1. What Comes From `motion-dom`

These are engine-owned pieces. Prefer using them directly over recreating them locally.

Runtime state and rendering:

- `HTMLVisualElement` / `SVGVisualElement`
- `VisualElement` state:
  - `latestValues`
  - render state
  - projection node
  - animation state
  - variant tree
  - presence context
- `createDomVisualElement()` in `package/src/component/create-dom-visual-element.ts` instantiates upstream visual elements.
- `createVisualState()` in `package/src/component/visual-state.ts` uses upstream scraping/build helpers from `motion-dom`.

Projection and layout:

- `HTMLProjectionNode`
- shared-layout stacks and promotion/relegation
- `nodeGroup()`
- scale correction for border radius and box shadow

Features and animation primitives:

- `setFeatureDefinitions()`
- `Feature`
- `animateVisualElement`
- gesture primitives used by hover/press/pan/in-view/focus features
- `animateMotionValue` from `motion-dom`, used inside drag behavior
- legacy animation controls subscriptions are feature-owned and must dispose the previous subscription before switching `animate` controls or leaving controls mode

Important consequence:

- For motion components, `motion-dom` owns the live animation/projection state.
- Local code should translate framework semantics into that engine, not duplicate the engine.

### 2. What We Translate From `motion/react` Into Solid

These are framework-owned behaviors. This repo has to translate them because `motion-dom` does not provide them.

Component factory and host wiring:

- `motion` proxy and intrinsic tag cache in `package/src/component/index.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leanderriefel/motion-solid](https://github.com/leanderriefel/motion-solid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
