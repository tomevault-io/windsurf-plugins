---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Reprise** (`symfony/reprise`) — a Symfony bundle that brings Webpack Encore's key features to modern bundlers: **Vite** and **Rsbuild/Rspack**. The bundle is a Composer package (PHP `src/`/`tests/` at the repo root); its JS side is the `@symfony/reprise` npm package, an [unplugin](https://github.com/unjs/unplugin) living under `assets/`. Full ESM, greenfield (early stage, most features are stubs).

**Design principle — do NOT re-implement what the bundler already does.** Vite and Rsbuild natively handle Sass/Less/PostCSS, TypeScript, code splitting, content hashing, source maps, minification, HMR, and the dev server. This plugin does NOT wrap or re-expose any of those (so no `enableSassLoader()`-style API from Encore). Its job is the **Symfony integration glue** that the bundlers do not provide — see below.

## Monorepo layout

The repo is a **Composer bundle** (`symfony/reprise`, PHP `src/`/`tests/` at the root) plus the **`assets/` npm package** (`@symfony/reprise`, the JS plugin and its tests), tied together by a pnpm workspace at the repo root.

## Commands

Package manager is **pnpm** (enforced via `packageManager` field). Node 22 (`.nvmrc`). The root `pnpm build`/`dev`/`test`/`lint`/`fmt` scripts run from the workspace root; `build`/`dev`/`test` delegate to the `assets` package, while `lint` (Oxlint) and `fmt` (Oxfmt) run at the root over the whole repo.

- `pnpm build` — delegates to `assets`, build via `tsdown` (bundles every `assets/src/*.ts` to `assets/dist/`)
- `pnpm dev` — delegates to `assets`, `tsdown -w`, watch/rebuild
- `pnpm lint` — `oxlint` at the root (config in `.oxlintrc.json`); `pnpm lint:fix` auto-fixes. `playground/`, `assets/test/fixtures/` and `docs/` are ignored (not library source)
- `pnpm fmt` / `pnpm fmt:check` — `oxfmt` at the root (config in `.oxfmtrc.json`); `fmt:check` is the read-only variant CI runs
- `pnpm test` — delegates to `assets`, run tests (vitest, scoped to `assets/test/` via `vitest.config.ts` so it never picks up `.references/` clones)
- `pnpm vitest run assets/test/index.test.ts` — run a single test file
- `pnpm vitest run -t "hi vitest"` — run a single test by name

### Playground (manual end-to-end verification)

`playground/` is a **full Symfony 7 PHP app** used to exercise the plugin against a real backend. It defines two entries (`app`, `admin`) and imports the plugin directly from `../assets/src` (Vite via `playground/vite.config.ts`, Rsbuild via `playground/rsbuild.config.ts`). `nodemon` rebuilds on `assets/src/**/*.ts` changes.

Run from the playground dir (the root `pnpm play` script is broken — it calls a nonexistent `dev` script):

- `npm -C playground run vite:dev` / `npm -C playground run vite:build`
- `npm -C playground run rsbuild:dev` / `npm -C playground run rsbuild:build`

## Architecture

Bundler-agnostic core + per-bundler adapters, all under `assets/src/`:

- `assets/src/core/` — pure, no bundler imports: `options.ts` (`normalizeOptions` + CDN guard + `resolvePublicPath`), `dev-server.ts` (`resolveDevOrigin`), `format.ts` (`buildEntrypoints`/`buildManifest` in the frozen v1 format), `emit.ts` (`writeSymfonyFiles`).
- `assets/src/collectors/` — turn a bundler's output into the shared `NormalizedGraph`: `vite.ts` (`bundleToGraph` from the Rollup bundle in build, `configToDevGraph` from the resolved config in serve) and `rspack.ts` (`statsToGraph` from the Rspack stats JSON).
- `assets/src/index.ts` — the `unpluginFactory` (Vite only now) + `createUnplugin` default export. Its `vite` hooks call the collectors + core: `config()` sets `base`/`outDir` and disables Vite's own manifest/publicDir copy; `generateBundle` emits the two files on build; `configureServer` writes the dev-flavoured files pointing at the dev-server origin.
- `assets/src/vite.ts` — one-line unplugin adapter `createVitePlugin(unpluginFactory)` (`@symfony/reprise/vite`).
- `assets/src/rsbuild.ts` — a **hand-written native `RsbuildPlugin`** (default export, `@symfony/reprise/rsbuild`), **not** an unplugin adapter. unplugin has no `createRsbuildPlugin`, and the Symfony integration needs Rsbuild-config-level control a raw Rspack plugin can't reach: `api.modifyRsbuildConfig` forces `tools.htmlPlugin = false` (no per-entry HTML) and disables the public-dir copy (output lives under `public/build`), and sets the output paths + dev origin; `api.onAfterCreateCompiler` taps `compiler.hooks.done` to run `statsToGraph` + core. It reuses the same core as the Vite path.
- `assets/src/types.ts` — public `Options` (`outputPath`, `publicPath`, `manifestKeyPrefix`, `devServerOrigin`) + the frozen `EntrypointsJson`/`ManifestJson`/`EntryFiles` shapes (`js`/`css`/`preload`/`dynamic`).

unplugin still earns its place for Vite and (upcoming) the Stimulus virtual module (universal `resolveId`/`load`, cross-bundler). Rspack is served exclusively through the native Rsbuild adapter — the raw Rspack unplugin adapter was dropped (Rsbuild is the supported Rspack layer).

## The Symfony integration contract (the core of this project)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [symfony/reprise](https://github.com/symfony/reprise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
