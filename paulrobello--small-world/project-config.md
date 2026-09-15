---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Table of Contents

- [What this is](#what-this-is)
- [Navigation quick start](#navigation-quick-start)
- [Running it](#running-it)
- [Testing](#testing)
- [Browser debugging](#browser-debugging)
- [Version](#version)
- [Release/deploy](#releasedeploy)
- [Architecture](#architecture)
- [Vibe](#vibe)
- [Conventions worth keeping](#conventions-worth-keeping)
- [Enhancement workflow](#enhancement-workflow)
- [graphify](#graphify)

## What this is

A single-page Three.js "terrarium" that procedurally generates a small floating-island world (biome, terrain, flora, creatures, birds, particles) from a 16-bit seed. Vite provides the dev server (HMR) and optimized production builds. Three.js and simplex-noise are npm packages, bundled and tree-shaken. Live site: https://small-world.pardev.net/

## Navigation quick start

Start with `README.md` for the user-facing product shape, then `index.html` for the static HUD markup (no importmap or CDN dependencies — everything is bundled by Vite), `main.js` for renderer/camera/animation-loop wiring, and `src/world.js` for the deterministic world-generation orchestration. Use `src/state.js` to understand shared mutable state before changing cross-cutting behavior. Feature work usually lands in one concern module under `src/` plus, when visible in the HUD/settings, the relevant `src/ui/` sub-module (`src/ui.js` is a thin entry point that just wires them up — see its Architecture bullet), `index.html`, and `style.css`. Portal work starts in `src/portal.js` (called from `src/world.js`); biome music in `src/music.js` (wired from `src/ui/settings-panel.js`); post-processing in `src/postfx.js`.

Source-of-truth docs/backlog:

- `CLAUDE.md` (this file) — architecture/conventions/gotchas for agents.
- `ideas.md` — curated enhancement backlog; completed items should be removed.

## Running it

`make dev` starts the Vite dev server in the foreground with hot reload on `http://localhost:2001`; `make dev-start` / `make dev-stop` / `make dev-restart` manage the same Vite server in the background. Edits to `main.js` / `src/*.js` / `style.css` / `index.html` are reflected instantly without a full reload when possible.

`make build` produces an optimized production bundle in `dist/` (minified, tree-shaken, content-hashed assets). `make preview` serves the built output locally.

```
make dev          # Vite dev server with HMR in the foreground
make dev-start    # Vite dev server with HMR in the background
make dev-stop     # stop the background dev server, or any process on PORT
make dev-restart  # restart the background dev server
make build        # production build → dist/
make preview      # preview production build
make lint         # ESLint over main.js and src/
make test         # all JS tests
make checkall     # all JS tests + lint + production build
make clean        # rm -rf dist
```

Runtime dependencies (three.js, simplex-noise) are installed via npm and bundled by Vite — they're no longer loaded from CDN. `node_modules/` is gitignored; run `npm install` before `make dev` or `make build`.

## Testing

`make test` runs the full suite: every `tests/*.test.mjs` file via plain `node` (no test runner/framework — each file asserts with `node:assert/strict` and exits non-zero on failure). `make checkall` runs `make test` followed by `make lint` and `make build`.

To run a single test:

```sh
node tests/determinism-seed.test.mjs
```

Most `*-static.test.mjs` files are **static invariant tests** — they import a module (e.g. `BIOMES` from `src/biomes.js`) or read source text and assert on its shape (a biome has a given flag, a constant has a given value, a code path exists). They catch config drift and accidental removal of a documented mechanism, but a pure refactor that preserves behavior can still break one — update the assertion to match the new shape rather than treating the failure as a regression. A smaller set of tests (no `-static` suffix, e.g. `caterpillar-trail-trim.test.mjs`, `fish-speed.test.mjs`, `portal-preview-pool-isolation-runtime.test.mjs`) exercise actual runtime behavior and should be treated as real regressions if they fail.

`tests/determinism-seed.test.mjs` is the guardrail for any change to world-gen: it verifies that the same seed reproduces the same world across regenerations. Anything that touches the seeded-PRNG window (see "The determinism trick" below) must be checked against this test — a subtle ordering change (a stray `Math.random()`, a re-ordered builder call) shifts the RNG stream for every seed without necessarily throwing an error.

## Browser debugging

When working from Codex Desktop, use the built-in browser debugging tools for local web app inspection, screenshots, console/network checks, and visual verification. Start the app with `make dev-start` if needed, open `http://localhost:2001` in the Codex browser tooling, and inspect the live app there.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulrobello/small-world](https://github.com/paulrobello/small-world) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
