---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo overview

Salty CSS is a build-time CSS-in-JS library. This is an Nx monorepo containing the core compiler, the per-framework integrations published to npm, and two example apps used as fixtures during development.

- `libs/core` (`@salty-css/core`) — the compiler, factories, parsers, generators, CLI, and shared types. Everything else depends on this.
- `libs/react`, `libs/next`, `libs/astro`, `libs/vite`, `libs/webpack` — framework integrations / bundler plugins. Each one is a separate npm package and a separate Nx project.
- `libs/eslint-config-core`, `libs/eslint-plugin-core` — ESLint config and plugin published with the library.
- `libs/cli` (`salty-css` on npm) and `libs/npm-create` — tiny wrappers that re-export `@salty-css/core/bin/main` so users can run `npx salty-css …` / `npm create salty-css`.
- `apps/react-testing`, `apps/astro-testing` — real apps used as fixtures to exercise the compiler end-to-end. `.saltyrc.json` points at these and `react-testing` is the `defaultProject`.

Source-of-truth docs live in the root `README.md`; `scripts/update-readmes.mjs` copies it into each published lib at release time. **Edit the root README, not the per-lib copies.**

When thinking of running tests, ALWAYS only run a currently supported "npm run" command from the repo root. Don't run `vitest`, `tsc`, or `nx` directly.

## Common commands

All commands run from the repo root. The repo uses Nx; equivalent `nx run …` forms work too.

- `npm run dev:react` — Vite dev server for the React fixture app (most useful for trying things out).
- `npm run dev:astro` — Astro dev server for the Astro fixture app.
- `npm run test:core` — vitest for `@salty-css/core` (the bulk of the logic).
- `npm run test:all` — vitest across core, vite, react, webpack, next.
- `npm run build:all` — runs `update-readmes` then builds every published package. Required before publishing.
- `npm run build:core` (and `build:react`, `build:vite`, …) — build a single package.
- `npm run pretty` — Prettier across the repo.
- `npx nx run core:lint` — lint a single project (the workspace doesn't define a root `lint` script).
- `npx nx run core:typecheck` — typecheck a single project.

Run a single test file: `npx nx run core:test -- src/parsers/parser.spec.ts` (or `npx vitest run libs/core/src/parsers/parser.spec.ts` from the repo root). Vitest is wired via the `@nx/vitest` plugin in `nx.json`; tests are colocated next to source as `*.spec.ts(x)`.

Publishing is automated via `npm run publish:all` / `publish:all-dev` (runs tests, builds, bumps versions with `lerna version --force-publish`, then publishes each package's `dist/` folder, prompting once for an npm OTP). Don't invoke these ever.

## Architecture

### The build pipeline (libs/core/src/compiler/salty-compiler.ts)

`SaltyCompiler` is the heart of the project. Bundler plugins instantiate one per project root and drive it. The flow:

1. **Locate the project.** `.saltyrc.json` (looked up by walking parent dirs) declares projects, their framework, and where `salty.config.ts` lives. `defaultProject` is the fallback.
2. **Collect "salty files."** Any file matching `*.(salty|css|styles|styled).(ts|tsx|...)` — see `saltyFileExtensions` in `libs/core/src/compiler/helpers.ts`. Files containing `defineX(` calls are also recorded as _config files_.
3. **Compile each salty file with esbuild** into `saltygen/js/<hash>-<contentHash>.js`, then dynamic-`import()` the result to read its exports. The compiler injects `globalThis.saltyConfig` (from `saltygen/cache/config-cache.json`) before the file runs, and rewrites `styled(X, …)` calls where `X` is imported from a non-salty file (so the import can be tree-shaken).
4. **Bucket exports by marker flags** set on the factory objects: `isMedia`, `isGlobalDefine`, `isDefineVariables`, `isDefineTemplates`, `isDefineFont`, `isClassName`, `isKeyframes`, or a `generator` for styled components.
5. **Emit CSS** into `<project>/saltygen/`:
   - `css/_variables.css`, `_reset.css`, `_global.css`, `_templates.css`, `_fonts.css` — global layers (always written, possibly empty).
   - `css/<component>-<hash>-<priority>.css` — per-component files.
   - `css/l_<priority>.css` — per-layer bundles when `importStrategy !== 'component'`.
   - `css/f_<file>-<hash>.css` — per-source-file bundles when `importStrategy === 'component'`.
   - `index.css` — top-level entry that declares `@layer reset, global, templates, fonts, l0…l8;` and imports the above. **This is what users `@import` from their app.**
   - `types/css-tokens.d.ts` — generated `VariableTokens` / `TemplateTokens` / `MediaQueryKeys` types consumed by user code via TS module augmentation.
   - `cache/config-cache.json` — merged config snapshot, also mirrored into the installed `@salty-css/core` package so other entrypoints can read it without re-running the compiler.

`generateCss()` is the full rebuild (used on `buildStart`); `generateFile(path)` is the incremental path used by HMR — it patches per-file CSS and re-injects into the right `l_<priority>.css` bundle without clearing `saltygen/`.

### Bundler plugins


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [margarita-form/salty-css](https://github.com/margarita-form/salty-css) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
