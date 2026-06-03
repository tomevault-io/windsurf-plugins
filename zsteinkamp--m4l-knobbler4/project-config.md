---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Knobbler4 is a Max for Live device that transforms a tablet/phone into an auto-labeling, multitouch parameter control surface for Ableton Live. It communicates with mobile apps via OSC (Open Sound Control) protocol and uses the Ableton Live API to interact with tracks, devices, and parameters.

## Development Environment

Code editing happens in a **VSCode Devcontainer** environment. The TypeScript source files in `src/` are edited within the container, which provides a Node.js 20 development environment with Prettier and ESLint extensions pre-configured.

**Opening the devcontainer:**
```bash
# From the repository root
dc  # Shell alias for 'devcontainer open .'
# Or manually:
devcontainer open .
```

**Container setup:**
- Based on `node:20` Docker image
- Mounts the repository to `/app` inside the container
- Automatically runs `npm run dev` (TypeScript watch mode) on startup
- Uses a Docker volume for `node_modules` to improve performance
- Pre-configured VSCode extensions: Prettier-ESLint, Prettier, Makefile Tools

**Workflow:** Edit TypeScript files in `src/` within the devcontainer → auto-compile via watch mode → compiled `.js` files appear in `Project/` → test changes in Ableton Live with the device reloaded.

**IMPORTANT — run node/build/test inside the devcontainer, never on the host.** The container is `m4l-knobbler4-node-1` (repo mounted at `/app`). Examples:
```bash
docker start m4l-knobbler4-node-1                      # if stopped
docker exec m4l-knobbler4-node-1 sh -c "cd /app && yarn build"
docker exec m4l-knobbler4-node-1 sh -c "cd /app && yarn tsc --noEmit"
docker exec m4l-knobbler4-node-1 sh -c "cd /app && yarn test"
```
The container usually runs `npm run dev` (tsc --watch) on startup, so saving a `src/*.ts` edit auto-recompiles into `Project/`.

**Run builds ONE AT A TIME — the devcontainer VM is memory-tight.** Docker Desktop's VM here is ~3.8 GiB and the `tsc --watch` dev chain idles near ~1 GiB, leaving only ~2.8 GiB headroom. A single `tsc` compile of this project (the generated 3000+-line `deviceParams.ts`) runs ~0.5–0.8 GiB, which is fine — but stacking several concurrent `yarn build`/`yarn tsc` invocations exhausts the VM, and the Linux OOM killer reaps `tsc --watch` (stops the container) and can destabilize `dockerd`/the bind-mount sync (symptoms: container exits with `did not receive an exit event`, or a *stale* `/app` mount where the container sees old source). Recovery: `docker restart` (re-syncs the mount); avoid `docker kill`. Prefer just saving the file and letting the watch recompile, or run one `yarn build` and wait for it. Optionally raise Docker Desktop memory to 6–8 GiB for headroom.

## Development Commands

### Build & Development

```bash
# Install dependencies
yarn install --frozen-lockfile

# Build TypeScript to JavaScript (compiles src/*.ts → Project/*.js)
yarn build

# Watch mode for development (auto-rebuild on changes)
yarn dev
```

### Code Quality

```bash
# Lint TypeScript files
npx eslint src/**/*.ts

# Format code with Prettier
npx prettier --write src/**/*.ts
```

## Architecture Overview

### TypeScript → Max/MSP Compilation Pipeline

TypeScript source files in `src/` compile to JavaScript in `Project/` directory. The compiled `.js` files are loaded by `[js]` objects in Max patches. TypeScript provides type safety and IDE support while Max's JavaScript engine executes the compiled CommonJS output.

**Key compilation setting**: `tsconfig.json` targets ES5 with CommonJS modules, output to `Project/` directory.

### Single-`[v8 knobbler]` architecture (current — May 2026)

The device is ONE `[v8 knobbler]` object (`src/knobbler.ts`) — the old `[v8 router]` and the per-feature `[v8]` objects were consolidated into it (only `[v8 k4-discovery]` stays separate).

`knobbler.ts` is the orchestrator/entry. It owns inbound OSC dispatch via a **route registry** (each feature module exports a `routes: Route[]` table; the entry merges them and calls `fn(...)` directly — no outlet fan-out) and builds an **`AppContext` (`ctx`)** that it hands to every module's `init(ctx)`. Modules reach siblings/services **through `ctx`** (`ctx.settings`, `ctx.osc`, `ctx.gotoTrack`, `ctx.gotoDevice`, `ctx.knobbler.bkMap`, …), never by importing each other.

**Why ctx and not imports:** Max `require()` does **not** cache modules — each file that imports another gets a *separate, dead* instance. So inter-module calls go through the single live instances the entry wired into `ctx`. The same tax means each module gets its own `utils` instance, so its `osc()` won't batch until `init` calls `setOscSink(ctx.osc)`.

**Adding a feature module (`k4-foo`):**

1. `src/k4-foo.ts` exports a `routes` table + an `init`:
   ```ts
   export const routes: Route[] = [{ prefix: '/foo', parse: 'val', fn: doFoo }]
   export function init(c: AppContext) {
     setOscSink(c.osc)   // required: own utils instance, see above
     ctx = c             // stash for ctx.settings / ctx.gotoTrack / etc.
     // observers, initial state push…
   }
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zsteinkamp/m4l-Knobbler4](https://github.com/zsteinkamp/m4l-Knobbler4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
