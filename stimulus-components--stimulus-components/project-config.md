---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A pnpm-workspace monorepo of small, independently-published [Stimulus](https://stimulus.hotwired.dev/) controllers. Each `components/<name>/` is its own npm package `@stimulus-components/<name>` with its own `package.json`, Vite build, and (often) tests. `docs/` is the Nuxt 3 documentation site published at stimulus-components.com.

This file is the contributor/agent guide for the repo. The Cursor skill `.cursor/skills/create-new-component/SKILL.md` is the step-by-step playbook for adding a component (covers package files, Vite config, docs registration, and demo wiring); follow it when creating one.

```
stimulus-components/
├── components/           # One publishable Stimulus controller per directory
│   └── <name>/
│       ├── src/index.ts  # Controller entry
│       ├── spec/         # Vitest tests (when present)
│       ├── vite.config.mts
│       ├── package.json
│       ├── README.md
│       └── CHANGELOG.md
├── docs/                 # Nuxt 3 documentation site
├── utils/                # Shared helpers (debounce, throttle, sleep)
├── .changeset/           # Release bumps (changesets)
├── pnpm-workspace.yaml   # Workspaces: docs, components/**
├── package.json
├── tsconfig.json
├── eslint.config.mjs
└── .prettierrc
```

## Commands (from repo root)

| Command            | Description                                         |
| ------------------ | --------------------------------------------------- |
| `pnpm install`     | Install all workspace deps                          |
| `pnpm run lint`    | `tsc --noEmit` (type-check) + ESLint                |
| `pnpm run lintfix` | Prettier `--write` + `eslint --fix`                 |
| `pnpm run test`    | Run all Vitest specs (`vitest --run`)               |
| `pnpm run docs`    | Start the Nuxt docs dev server (`pnpm -C docs dev`) |

Run a single test file: `pnpm exec vitest --run components/<name>/spec/index.test.ts`. Build one component: `pnpm -C components/<name> run build` (Vite lib build, then `pnpm run types` emits `dist/types/`). Build every component: `pnpm run build:components`. CI runs lint, test, the component builds, and `docs generate` on every PR (`.github/workflows/ci.yml`); `release.yml` is separate.

## Architecture notes

- **Each component is isolated.** A component depends only on `@hotwired/stimulus` (peer dep, `^3`) plus, when needed, `stimulus-use` or a specific lib (e.g. `sortablejs`, `chart.js`). Don't add root-level deps for a single component, and don't bump the Stimulus `^3` range without checking every component and the docs.
- **Controller shape** (`src/index.ts`): one default-exported class extending `Controller`, `static targets`/`static values` declared with TypeScript `declare`, lifecycle in `connect()`/`disconnect()` (use `initialize()` only to bind event-handler methods). Class is PascalCase; package dir, controller identifier, and `data-controller` are all the same kebab-case; build output is `stimulus-<name>.mjs` / `.umd.js`.
- **Shared helpers** live in root `utils/` (`debounce`, `throttle`, `sleep`), imported relatively as `"../../../utils"` from inside `components/<name>/src/`.
- **Docs wiring is manual.** A new component must be registered in `docs/plugins/stimulus.client.ts` (`application.register("<name>", PascalName)`), added to `docs/package.json` as `"workspace:*"`, given a `docs/content/docs/stimulus-<name>.md` page (sidebar is auto-built from this dir, sorted by title), and a `docs/components/content/Demo/<PascalName>.vue` demo referenced as `:<name>` in the doc page.
- **Per-component `package.json`** declares `main` (UMD), `module` (`.mjs`), `types`, an `exports` map, at least `@hotwired/stimulus: ^3` in `peerDependencies`, `"files": ["dist"]`, `"sideEffects": false`, and a `"prepack": "pnpm run build"` hook. Keep these paths in sync with what Vite actually emits — a stale `module`/`types` path fails silently until someone installs the package.
- **The `exports` map is a strict allowlist.** Each package exposes `.` (with `types`/`import`/`require` conditions, `types` first), `./src`, and `./package.json`. The `./src` subpath is not optional: the docs app imports `@stimulus-components/<name>/src` to consume the TypeScript sources directly, and dropping it from `exports` breaks the docs build. Keep the legacy `main`/`module`/`types` fields alongside it for older bundlers.
- **Docs styling** is Tailwind 4, configured CSS-first in `docs/assets/css/tailwind.css` (`@theme`, `@custom-variant`, `@source`) and wired via the `@tailwindcss/vite` plugin in `docs/nuxt.config.ts`. There is no `tailwind.config.ts`.

## Style & types


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stimulus-components/stimulus-components](https://github.com/stimulus-components/stimulus-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
