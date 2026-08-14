---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Motion Script is an open-source motion design tool (inspired by Manim) for
authoring animations as TypeScript/JSX "scenes" and rendering them in the
browser via Skia/CanvasKit. Scenes are generator functions: `yield*`-ing a
tween hands control back to the engine to advance time.

## Commands

pnpm workspace + Turborepo. Node.js LTS, pnpm (pinned via `packageManager` in
root `package.json`).

```bash
pnpm install
pnpm build            # turbo run build, all packages, topological order
pnpm build:lib        # build everything except @motion-script/site (used by CI before tests)
pnpm test             # pnpm -r test, every package's vitest suite
pnpm clean            # turbo run clean
```

Per-package (most library packages support `dev`, `build`, `lint`, `typecheck`, `test`):

```bash
pnpm --filter @motion-script/core dev          # watch build
pnpm --filter @motion-script/core test         # vitest, watch mode
pnpm --filter @motion-script/core test -- run  # single run, no watch
pnpm --filter @motion-script/core lint
pnpm --filter @motion-script/core typecheck
```

To run a single test file, pass it through the filter to vitest, e.g.
`pnpm --filter @motion-script/core test -- run src/tween/tween.test.ts`.

`@motion-script/web`'s tests run in a real headless Chromium via
`@vitest/browser-playwright` — install the browser once with
`pnpm --filter @motion-script/web exec playwright install --with-deps chromium`.
Its tests import `@motion-script/core`'s **built** `dist/` (package `exports`
point there), so run `pnpm build:lib` (or at least build `core`) before
`pnpm --filter @motion-script/web test` on a fresh checkout.

E2E (visual regression, Playwright, in `packages/e2e`):

```bash
pnpm test:e2e         # pnpm --filter @motion-script/e2e run test:e2e
pnpm e2e:stable       # (re)pack the committed "stable" baseline tarballs
pnpm e2e:shoot:lib    # render scenes with the current branch's lib build
pnpm e2e:shoot:stable # render scenes with the stable baseline
pnpm e2e:compare      # pixel-diff lib vs stable
```

CI (`.github/workflows/ci.yml`) runs two independent jobs: `pnpm test` after
`pnpm build:lib` (unit tests), and a Dockerized render-and-pixel-diff of every
`packages/e2e` scene against the committed stable baseline vs. the branch's lib
build (e2e-visual).

### Visually verifying a change with `ms screenshot`

`@motion-script/cli` (`ms`) is a devDependency of every example/test project
(`packages/template`, `packages/e2e`). It boots a real
Vite dev server (via `@motion-script/vite-plugin`) headlessly with Playwright
Chromium, so it needs the libraries built at least once first — run
`pnpm build:lib` (or rebuild the specific package you touched) on a fresh
checkout or after editing `core`/`web`/`player`.

```bash
pnpm --filter @motion-script/template exec ms list             # scene names in that project
pnpm --filter @motion-script/template exec ms screenshot last   # last frame, combined timeline
pnpm --filter @motion-script/template exec ms screenshot first --split   # frame 0 of every scene, one file each
pnpm --filter @motion-script/template exec ms screenshot 2.5s --scenes intro
pnpm --filter @motion-script/template exec ms clear             # delete everything under out/
```

`<when>` is a frame index (bare integer), a time (`2.5` or `2.5s`), or
`first`/`last`; see `ms --help` for the rest of the flags (`--scale`,
`--format`, `--out`). Files land under `<project>/out/screenshots/` (e.g.
`packages/template/out/screenshots/intro_75.png`) — after capturing, use the
Read tool on that PNG path to actually look at the frame rather than assuming
the render is correct. This is the fast way to confirm a node/attribute/tween
change renders as intended without opening the interactive player. `ms export`
(same driver) renders a scene to MP4 instead, for checking motion over time
rather than a single frame.

### Build orchestration — read before touching a package's build config

**Turbo owns build ordering, not TypeScript.** Each package builds with plain
`tsc -p tsconfig.build.json` (project mode), never `tsc -b` (build mode):
`tsc -b` would walk `references` and rebuild dependencies itself, racing with
Turbo's own `dependsOn: ["^build"]` dependency builds and corrupting `dist/`
on a clean build. `tsc -p` only compiles the current package, reading
dependencies' already-built `dist/*.d.ts` (which Turbo guarantees exist first).

Conventions to preserve when adding/editing a package:

- **`references` in `tsconfig.json` are editor-only**, and must point at the
  dependency's `tsconfig.build.json` (e.g. `{ "path": "../core/tsconfig.build.json" }`),
  never the bare directory — that resolves to the test-inclusive config and
  diverges from what actually gets built.
- **Two tsconfigs per package**: `tsconfig.json` (test-inclusive; used by the
  editor and `typecheck`) and `tsconfig.build.json` (extends it, excludes
  tests; what `build` and consumers' `references` point at).
- **`tsBuildInfoFile` lives inside `dist/`** (`"dist/tsconfig.build.tsbuildinfo"`)
  so it's covered by Turbo's `dist/**` output cache; each package's `files`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [motion-script/motion-script](https://github.com/motion-script/motion-script) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
