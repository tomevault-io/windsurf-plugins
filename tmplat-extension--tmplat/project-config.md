---
trigger: always_on
description: Guidance for AI coding agents (and human contributors) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (and human contributors) working in this repository.

## Project Overview

**tmplat** (`package.json` name: `tmplat`) is a Google Chrome extension that quickly and easily copies info
about the current page using a simple, unique template system.

This repository is **mid-migration** from a legacy codebase (internally still referred to as "Template") to the
new "tmplat" codebase. Both old and new code currently coexist in `src/`. Understand which "world" a file
belongs to before editing it.

| Aspect            | Legacy ("Template")                                                                                                          | New ("tmplat")                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Language          | CoffeeScript (`.coffee`)                                                                                                     | TypeScript (`.ts` / `.tsx`)                                                            |
| UI framework      | Bootstrap 2/3 + jQuery + Backbone + Mustache (`.html` templates)                                                             | React + MUI (Material UI) + Emotion                                                    |
| Manifest          | Manifest V2 (background page, `browser_action`, etc.)                                                                        | Manifest V3 (`src/manifest.json` — service worker, `action`)                           |
| Build tool        | Grunt (no longer present in this repo)                                                                                       | [rolldown](rolldown.config.mjs) via `rolldown.config.mjs`                              |
| Dependency style  | Vendored libraries in `src/vendor/` (jQuery, Backbone, Underscore, Async, Mustache, purl.js, date-ext, md.min.js, Bootstrap) | npm packages (React, MUI, inversify, luxon, numeral, lodash.\*, tmplat-mustache, etc.) |
| DI / architecture | Ad-hoc globals/singletons                                                                                                    | `inversify` DI container (see `src/lib/common/di.ts`)                                  |

### Signs a file is legacy vs new

- Legacy: `src/lib/*.coffee` (`background.coffee`, `content.coffee`, `options.coffee`, `popup.coffee`,
  `utils.coffee`), `src/options.legacy.html`, `src/popup.legacy.html`, and anything under `src/vendor/`.
- New: everything under `src/lib/**/*.ts` and `src/lib/**/*.tsx`, especially the `ui/` (React+MUI),
  `worker/` (MV3 service worker), `content/` (MV3 content scripts), `template/`, `tab/`, `oauth/`,
  `context-menu/`, `analytics/`, and `common/` directories, plus `src/options.html`, `src/popup.html`,
  `src/migrate.html`.
- `src/migrate.html` / `src/lib/ui/migrate/` exists specifically to help migrate a user's data/settings from the
  legacy storage format to the new one — this is a real, permanent feature of the migration, not a scratch file.

See [MIGRATION-GAPS.md](MIGRATION-GAPS.md) for an evidence-backed list of the known functional gaps between the
legacy CoffeeScript implementation and the new TypeScript one (non-UI). Consult it before assuming a feature has
already been ported.

**Do not "fix" legacy CoffeeScript files by rewriting them in TypeScript unless explicitly asked to migrate that
specific piece of functionality.** When asked to migrate functionality, prefer creating idiomatic new TypeScript
(strict mode, DI via `inversify`, React function components with MUI) rather than a literal line-by-line port.
Once a piece of legacy functionality has an equivalent under `src/lib/**/*.ts(x)`, the corresponding `.coffee`
file/logic should eventually be removed — check whether that has already happened before assuming legacy code is
still in use.

## Tech Stack (new codebase)

- **Language**: TypeScript (strict mode), targeting `es2022`, compiled/bundled with type-stripping via rolldown
  (see `tsconfig.json`). Path alias `extension/*` maps to `src/lib/*`.
- **UI**: React 19 + MUI (`@mui/material`, `@mui/icons-material`, `@mui/x-data-grid`) + Emotion for styling. SCSS
  is still used in a couple of places (`src/scss/migrate.scss`, `src/scss/popup.scss`) and compiled by a custom
  Sass plugin in `rolldown.config.mjs`.
- **DI**: `inversify` (re-exported through `src/lib/common/di.ts`, which also imports `reflect-metadata`).
  Decorators (`experimentalDecorators`/`emitDecoratorMetadata`) are enabled for this reason.
- **Build**: [rolldown](https://rolldown.rs) configured in `rolldown.config.mjs`. Each entry point in the
  `entries` map produces an IIFE bundle under `dist/temp/...`. The config also has custom plugins to: copy static
  assets/images/vendor files, compile SCSS to CSS, minify/copy `_locales` i18n files, copy `manifest.json`/other
  JSON (injecting the version from `package.json`), and copy root HTML files.
- **Manifest**: Manifest V3 (`src/manifest.json`) — service worker background (`lib/worker/background.js`),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmplat-extension/tmplat](https://github.com/tmplat-extension/tmplat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
