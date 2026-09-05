---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Skip — an Angular 21 (zoneless, signals, new control flow) Signal K marine instrument panel, `@halos-org/skip`. It's a Signal K webapp the SK server serves at the path `/@halos-org/skip/` (its own npm package name). Skip **originated as a fork of `mxtommy/kip`** (never a GitHub fork object) but is now an **independent project** — KIP is not an upstream we track, sync with, or treat as authoritative. What set Skip apart from the start was its auth + profiles work (below), which KIP did not accept; it has diverged further since.

**KIP is not "upstream."** There is no goal of staying rebaseable on or in sync with `mxtommy/kip`, and no obligation to mirror its structure. If a specific KIP change is ever worth having, treat it as borrowing an idea from a separate project (reimplement it to fit Skip), not as merging from an upstream. Design decisions weigh only correctness and our own maintenance cost.

## Commands

- `npm run dev` — dev server (serve-path `/@halos-org/skip/`).
- `npm run build:prod` / `npm run build:dev` — Angular app build (output → `public/`).
- `npm test` — full unit suite, headless (vitest via `@angular/build:unit-test`). `npm run test:interactive` for watch mode.
- `npm run lint` — ESLint (flat config). `@typescript-eslint/no-explicit-any` is an **error**, and `no-unused-vars` does **not** ignore `_`-prefixed params — drop unused params, don't underscore them.
- `npm run generate:widget` — schematic that scaffolds a Host2 widget (preferred over hand-writing one).
- `./run` — standard HaLOS dispatcher (`./run help` for commands): `build`, `test`, `lint`, `ci` (lint + snc + tests, the CI gate), `bumpversion patch|minor|major`.

Node: the app builds on Node 20+, but only **Node 24 is CI-verified** (`run-tests` and the npm-publish job run Node 24 — the 20/22/24 matrix was dropped when the bespoke `ci.yml` was replaced).

CI is the standard HaLOS triad: `pr.yml` → shared `pr-checks` (with `skip-lintian`), `main.yml` → shared `build-release` in npm-only mode (`build-deb: false`; cuts a draft stable release), `release.yml` → `npm publish` when that release is published. `VERSION` is the source of truth, synced to `package.json` by `./run bumpversion`. In npm-only mode a `+N`-only merge (no `VERSION` change) cuts a GitHub release that publishes nothing to npm — bump `VERSION` to ship a new version.

## strictNullChecks (enabled)

`strictNullChecks` is **enabled** for the whole app in `tsconfig.json` (issue #6, closed). The TypeScript compiler and Angular's `strictTemplates` now enforce null-safety across sources, component templates, and specs — a null-safety regression fails the build directly.

- `npm run snc` — the deterministic type-check gate: `tsc -p tsconfig.strict.json` (production `src/**/*.ts`) **and** `tsc --noEmit -p src/tsconfig.spec.json` (specs). Specs are checked here explicitly because the vitest builder's own spec type-checking is cache-sensitive and not reliable in CI.
- The CI gate is `npm run ci` = `lint` → `snc` → `test:headless` → `test:mcp-schema`. `snc` enforces null-safety in production sources and specs; the app/test build (Angular `strictTemplates`) enforces it in component templates. All three surfaces are covered.

The former **betterer ratchet** that migrated the codebase to zero file-by-file was retired at the flip; `.betterer.*` and `tsconfig.betterer.json` are gone — do not reintroduce them. The remaining `strict`-family flags (`strictPropertyInitialization`, `strictFunctionTypes`, `noImplicitAny`, `noImplicitReturns`, `noFallthroughCasesInSwitch`, …) are **not** yet enabled; their measured error count is tracked in the rest-of-strict follow-up issue.

## TypeScript only

Skip strives to be **100% TypeScript**. No new JavaScript source files are allowed, and any existing JavaScript source that a change touches must be converted to TypeScript as part of that same change — not left as JS with a follow-up promise. The one exception is tool/build config that is conventionally JavaScript (e.g. `eslint.config.js`); those may stay JS.

## Webapp-only (no bundled server plugin)

The Skip package must remain **webapp-only** — the `signalk-webapp` keyword only, no `signalk-node-server-plugin`, no `main`, no bundled server plugin. Bundling an in-process server plugin forces a Signal K server restart on every webapp update (the server loads plugin code into its process at boot). The Freeboard-SK integration lives in the separate `@halos-org/skip-freeboard-panel` package, pulled in as a runtime dependency. This is machine-enforced by the `run-tests` CI action, which fails if the publish tarball ships any `plugin/`.

## Performance and freeze measurement (perf-harness/)

Self-contained freeze/jank harness (own `package.json`, never touches app deps): builds the **real production bundle**, drives it in headless Chromium at **10× CPU throttle** (Pi-class HaLOS target) against a **mock Signal K server** on one origin. Use it for perf-sensitive changes to widgets, rendering, or the data pipeline, and for before/after numbers on freeze/jank fixes. Full operator docs: `perf-harness/README.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [halos-org/skip](https://github.com/halos-org/skip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
