---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

ReactUnity: a React renderer that draws UI inside Unity3D (UGUI, UIToolkit, and the Unity Editor) without a DOM. This is a **monorepo formed by merging ~10 previously separate ReactUnity repos** (`renderer`, `scripts`, `material`, `create`, `core`, `jint`, `quickjs`, `clearscript`, `docs`, `tests`, `full-sample` (now `kitchen-sink/`), `samples`) with their histories preserved — see the `chore: merge X into Y` commits. Much of the layout only makes sense in that light, and the workflow files under `.github/workflows/` carry detailed comments explaining why each piece is shaped the way it is. Read those before changing CI.

## Two package universes

| | npm (`packages/*`) | UPM / Unity (`unity/*`) |
|---|---|---|
| Members | `@reactunity/renderer`, `scripts`, `material`, `create` | `com.reactunity.core`, `jint`, `quickjs`, `clearscript` |
| Consumed by | the user's React app | the user's Unity project |
| Published by | `.github/workflows/release-npm.yml` (Tegami + npm OIDC) | `.github/workflows/release-upm.yml` (manual dispatch, orphan branch per package) |

All eight share **one version number** (currently `0.22.0`). Tegami bumps `packages/*`; the `syncUnityVersions` plugin in [scripts/tegami.mts](scripts/tegami.mts) copies that version into the four `unity/*/package.json` manifests.

**`unity/*/package.json` files are UPM manifests, not npm manifests.** Their `dependencies` are Unity package names (`com.unity.editorcoroutines`). They are deliberately excluded from the pnpm workspace — adding them would send pnpm to the npm registry looking for Unity packages.

## pnpm workspace membership

Defined in [pnpm-workspace.yaml](pnpm-workspace.yaml), which documents every inclusion and exclusion. Members are: `packages/*`, `unity/core/.react/*` (three React apps embedded in the core Unity package; the `.react` dot-prefix hides them from Unity's asset importer), `kitchen-sink/react`, and `docs`.

Explicit non-members: `samples/**` (rotted — React 19 alongside React-18-capped redux deps), `packages/create/scaffold/**` (a template whose deps are placeholders), and anything under `Library/` or `PackageCache/` (Unity's own package cache is full of `package.json` files).

`overrides` in the same file force every `@reactunity/*` request to resolve to the workspace copy — without it pnpm downloads registry copies of packages that exist right here, and hard-fails once the local version is one npm has never seen.

## Commands

Node >= 26 (`.node-version` pins 26), pnpm 11.17.0 via `packageManager`.

```bash
pnpm install
```

```bash
pnpm build
```
Builds `packages/*` only, topologically (`material` needs `renderer`'s `dist`). [tsdown](https://tsdown.dev) is the bundler — one `tsdown.config.ts` per package, and `tsc` never emits. Output is bundled per entry rather than mirroring `src`, and targets **ES2015**, which is rolldown's floor (it cannot emit ES5). The config comments cover the externals.

```bash
pnpm typecheck
```
`tsc --noEmit` across the workspace, and the only thing that type checks — neither tsdown nor webpack does. CI runs it as its own step.

```bash
pnpm check
```
Biome lint + format check over the whole repo — this is what CI runs. `pnpm lint` and `pnpm format` are the narrower variants. Husky + lint-staged run `biome check --write` on commit.

Per-package work uses pnpm filters:

```bash
pnpm --filter @reactunity/renderer build
```

```bash
pnpm --filter @reactunity/material watch
```

### Running a React app against Unity

Each app (`kitchen-sink/react`, `unity/core/.react/*`) uses `react-unity-scripts`, a CRA fork:

```bash
pnpm --filter reactunity-kitchen-sink start
```

`start` runs a dev server with HMR that Unity connects to (and serves a browser previewer at the port). `build` emits to `BUILD_PATH` — by default `../Assets/Resources/react`, overridden per app in its `.env`. `react-unity-scripts start --test` swaps the entry point to `test.ts`. See [packages/scripts/README.md](packages/scripts/README.md) for the full env-var surface (`FILENAME`, `BUILD_PATH`, `JSX_IMPORT_SOURCE`, …).

### The documentation site

`docs/` is an [Astro](https://astro.build) site (it was a Next 12 fork of react.dev until the migration; the shape and MDX component set are still React's). `pnpm --filter react-website start` serves it on port 4321, `pnpm --filter react-website build` writes `docs/dist`. There is no type-check command for it -- see below. [docs/README.md](docs/README.md) explains how a page is assembled; the parts worth knowing before editing it:

- Pages are `.mdx` files in `docs/src/content`, wired up by `docs/src/content.config.ts` and rendered by `docs/src/pages/[...slug].astro`, which also owns the element-name → component map. Navigation, titles and prev/next links come from `sidebarLearn.json` / `sidebarReference.json`, not frontmatter. `layout:` is *not* usable in frontmatter — Astro treats it as a component to import.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ReactUnity/core](https://github.com/ReactUnity/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
