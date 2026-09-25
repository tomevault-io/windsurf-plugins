---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

OpenCiv is a browser-based, Civ-5-inspired turn-based strategy game. It's a monorepo with two independent TypeScript projects, `client/` and `server/`, that talk to each other exclusively over a raw WebSocket protocol (no REST API). There is no shared types package between them — see "Client/server split" below.

## Commands

Run all of these from the repo root unless noted.

```bash
npm run install-all      # npm install at root, client/, and server/ (all three are separate node_modules)
npm start                 # boots server (ws://localhost:2000) and client dev server (http://localhost:1234) together
```

Client (`cd client`):
```bash
npm run dev                                    # Parcel dev server with HMR
npm run build                                  # Parcel production build -> client/dist
npx tsc --noEmit                                # typecheck (matches what VSCode/Parcel see)
```

Server (`cd server`):
```bash
npm start                                       # ts-node-dev, respawns on change
npm start -- --help                             # list the game options that can be set at launch
npm start -- --no-allowBarbarians --numCityStates=0   # start with game option overrides
npm test                                        # full Jest suite
npx jest tests/unit/Unit.test.ts                # single test file
npx jest -t "test name"                         # single test by name
npx tsc --noEmit                                # typecheck
```

CI typecheck — **do not use plain `tsc` for this**, see "The `tsconfig.typecheck.json` split" below. There is no root `tsconfig.typecheck.json`: each project has its own, so this is the one command that does *not* run from the repo root. Run it once per project, the way `.github/workflows/build.yml` does:
```bash
cd client && npx tsc -p tsconfig.typecheck.json --noEmit
cd server && npx tsc -p tsconfig.typecheck.json --noEmit   # from the repo root again
```
Pass `--noEmit` when running it by hand. Neither project's config sets it, so CI's bare `tsc -p tsconfig.typecheck.json` emits JS — fine in a throwaway checkout, litter in yours.

### Don't run `npm run build` to verify a change

`npx tsc -p tsconfig.typecheck.json --noEmit`, run from `client/`, is the gate CI enforces and is what you should run after editing client code. A Parcel production build takes many minutes from a cold cache and reports nothing the typecheck didn't already catch — it is not a smoke test. Only run it when the bundler output itself is what's in question (a Parcel config/asset-resolution change).

If you do run it and it exceeds the tool timeout, it gets backgrounded and **killing the task does not kill Parcel** — the npm wrapper dies and the `parcel build` child keeps running, holding an LMDB lock on `client/.parcel-cache`. Every later `rm -rf .parcel-cache` then fails with `Device or resource busy`, and retrying the build just adds another orphan. Recover by stopping the stray processes first:
```bash
node scripts/kill_dev_processes.js   # stops orphaned parcel/ts-node-dev processes
```

### The dev server must not watch `client/dist` — keep `--watch-ignore dist`

Parcel's watcher ignores only `.git`, `.hg` and the cache dir (`getWatcherOptions` in `@parcel/core/lib/RequestTracker.js`) — **not** the dist dir. `client/dist` sits inside the watched project root, so Parcel sees its own output as source changes and re-enters packaging, which rewrites `dist`, which fires more events. With ~148 emitted files this self-loop wins the race often: measured **8 of 15** cold starts hung forever, versus **0 of 15** with `--watch-ignore dist` in the client's `dev` script. Don't remove that flag.

A hung start looks like this — note `dist` is already fully written and the build still never finishes:
```
Building...
Bundling...
Packaging & Optimizing...     <- repeats forever, no "✨ Built in"
```
A few `Packaging & Optimizing...` lines in one healthy build are normal: Parcel's progress reporter has no TTY under `concurrently`, so each progress update prints its own line instead of overwriting.

Don't add `--no-cache` to the dev script. It doesn't stop Parcel writing the cache, it only stops it *reading* it, so every start pays a full cold rebuild (~1.2s vs ~100ms warm) and the cache still grows.

Orphaned dev servers are a separate problem: killing the npm/npx wrapper leaves the real `parcel` child alive, and freeing the port doesn't help because the orphan no longer holds one. `npm start`'s `prestart` runs `scripts/kill_dev_processes.js`, which kills the processes themselves.

Manual/E2E test flow (root):
```bash
npm run test:e2e                                # or: npm run test:e2e -- --scenario=CitySettlement
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RyanGrieb/OpenCiv](https://github.com/RyanGrieb/OpenCiv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
