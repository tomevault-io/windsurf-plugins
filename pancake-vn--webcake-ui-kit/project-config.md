---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`webcake-ui-kit` is a Vue UI library that ships **raw `.vue` files** (no build step at the library level) and is consumed by both Vue 2 and Vue 3 apps. The package's `main` is `src/index.js` and `package.json` only includes `src/` in `files`. The "build" script is intentionally a no-op:

```
"build": "echo 'No need to build - shipping raw SFC'"
```

This shape is the central design constraint — every authoring decision flows from it.

## Dual-compat is the whole product

Components must compile under both Vue 2.7 (Options API era, vue-template-compiler) and Vue 3.4 (`@vitejs/plugin-vue`). Concretely:

- **Pure Options API only.** No `<script setup>`, no `setup()`, no Composition API imports. Per `package.json` description.
- **Single root element** in `<template>` (Vue 2 has no fragments).
- **Always declare `emits`** on the component — Vue 2.7 accepts and ignores it; Vue 3 needs it for `$attrs` separation.
- **Avoid Vue-3-only features** (`<Teleport>`, `<Suspense>`, `v-model:arg`, multi v-model) and **Vue-2-only features** (filters, `.native`, `$listeners`, `Vue.set`/`Vue.extend`).
- **`$attrs` divergence:** Vue 3's `$attrs` includes `class`/`style`, Vue 2's does not. Don't read these off `$attrs`.

Four project-scoped skills encode the workflow:

- `.claude/skills/vue-dual-component/SKILL.md` — full DO/DON'T list + SFC skeleton. Auto-loads on edits to `src/components/*.vue` or `src/index.js`.
- `.claude/skills/figma-to-component/SKILL.md` — orchestrates Figma → SFC: composes the Figma MCP (`mcp__plugin_figma_figma__*`) with the dual-compat rules, writes the unit spec via `component-test`, and delegates the post-write quality gate to `component-review`. Triggers on prompts like "Implement this design from Figma. @<figma.com URL>".
- `.claude/skills/component-test/SKILL.md` — dual-compat unit-test author. Produces `tests/<Name>.spec.js` that runs against Vue 2.7 and Vue 3.4 via the two Vitest configs. Triggers on "viết test / thêm test / test component", or when a new component lacks a matching spec. Invoked as Phase 2 step 9 by `figma-to-component`.
- `.claude/skills/component-review/SKILL.md` — standalone quality gate. Runs file-layout audit, dual-compat static check, CSS/token audit, wiring audit (index.js + playgrounds + storybook), and `npm run test:build`. Triggers on "review", "kiểm tra", "audit", or as Phase 3 of `figma-to-component`.

## Adding a component

A new component touches **ten files** in one change — anything less and `test:build` / `test` are meaningless:

1. `src/components/<name>/<Name>.vue` — the SFC. Single-root template, Options API, validated props, declared `emits`, BEM `ui-<name>--<modifier>`. **No inline `<style>` block** — link the CSS file via `<style src="./<name>.css" scoped></style>`.
   1b. `src/components/<name>/<name>.css` — the actual CSS rules (folder name lowercase, Vue file PascalCase, CSS file lowercase).
2. `src/index.js` — named export, alphabetical: `export { default as <Name> } from './components/<name>/<Name>.vue'`.
3. `src/styles/*.css` — add any missing semantic tokens (most are already there).
4. `playground-vue3/src/main.js` — register globally with `app.component(...)`.
5. `playground-vue2/src/main.js` — register globally with `Vue.component(...)`.
6. `playground-vue3/src/App.vue` — append a showcase `<section>` exercising every variant.
7. `playground-vue2/src/App.vue` — append the **same** section (keep the two App.vue in sync).
8. `storybook-vue3/stories/<Name>.stories.js` — CSF 2 file. Required exports: `Primary`, `AllVariants`, `Matrix` (when 2+ prop axes), one slot story, and `FocusVisible` (with `tabindex="0"`) if the component has `:focus-visible` styling. Storybook auto-loads `src/styles/index.css` via `.storybook/preview.js` — don't import per-story.
9. `tests/<Name>.spec.js` — dual-compat unit spec. Import from `'../src/index.js'`, use `mount` from `./_utils.js`. Cover smoke render, each enum prop, each emit (positive + disabled negative), each named slot, boolean state classes, and v-model emits. See `.claude/skills/component-test/SKILL.md`.

Then `npm run test:build` for compile-check, `npm test` for the dual-compat unit suite, `dev:vue3` / `dev:vue2` for visual, and `cd storybook-vue3 && npm run storybook` for the docs view.

The skill at `.claude/skills/figma-to-component/SKILL.md` automates steps 1–9 from a Figma URL and runs `npm test` + `test:build` itself before handing off.

## Commands

All commands run from the repo root unless noted.

```
npm run dev:vue3        # Vite playground (port 8001) — Vue 3.4
npm run dev:vue2        # webpack-dev-server (port 8080) — Vue 2.7
npm run preview         # both dev servers in parallel via concurrently — open both ports side-by-side
npm run dev             # bare Vite at root (rarely used; no entry HTML)
npm run build           # no-op, prints reminder

npm run build:vue3      # production build of playground-vue3 (compile check)
npm run build:vue2      # production build of playground-vue2 (compile check)
npm run build:storybook # build storybook-vue3 (compile check for stories)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pancake-vn/webcake-ui-kit](https://github.com/pancake-vn/webcake-ui-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
