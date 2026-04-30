---
trigger: always_on
description: **Reality:** The product is **midee** (`package.json` → `"name": "midee"`). The repo directory on disk is often **`pianoroll`** — same codebase. Everything user-facing is a **static Vite SPA**: MIDI, audio, Pixi canvas, and MP4 export run in the **browser only**; there is no app server for core features (deploy is static assets + optional analytics keys).
---

# Agent instructions — midee (pianoroll)

**Reality:** The product is **midee** (`package.json` → `"name": "midee"`). The repo directory on disk is often **`pianoroll`** — same codebase. Everything user-facing is a **static Vite SPA**: MIDI, audio, Pixi canvas, and MP4 export run in the **browser only**; there is no app server for core features (deploy is static assets + optional analytics keys).

## Commands (from `package.json`, repo root)

```bash
npm install
npm run dev          # vite → default http://localhost:5173
npm run check        # typecheck && biome check src && vitest run
npm run typecheck    # tsc --noEmit  (see tsconfig: include is src/** only)
npm run lint         # biome check src
npm run lint:fix     # biome check --write src
npm run format       # biome format --write src
npm run test         # vitest run  (jsdom; see vite.config.ts test.*)
npm run build        # tsc && vite build
# postbuild (automatic after build): node scripts/build-content.mjs, build-og.mjs,
# stamp-sitemap.mjs, check-links.mjs — static content / SEO, not runtime app logic
```

Prefer `npm run check` before you call a change “done”.

## Tooling (grounded in config files)

| Piece         | Where it’s defined        | What to know                                                                                                                              |
| ------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| TypeScript    | `tsconfig.json`           | `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `jsxImportSource: solid-js`, `include: ["src"]` only                  |
| Lint / format | `biome.json` + scripts    | `npm run lint*` / `format` operate on **`src/`** (matches `"lint": "biome check src"`). Biome `files.includes` also lists `src/**/*.css`. |
| Tests         | `vite.config.ts` → `test` | `environment: 'jsdom'`, `include: ['src/**/*.test.ts', 'src/**/*.test.tsx']`, `setupFiles: ['./vitest.setup.ts']`                         |
| Bundler       | `vite.config.ts`          | `vite-plugin-solid`; **`resolve.alias.events → 'events'`** because `@tonejs/piano` pulls Node’s `events` — needed for browser builds      |
| Env           | `src/env.ts`              | `@t3-oss/env-core` + Zod; **client** vars use `VITE_` prefix (`clientPrefix`)                                                             |

Dependency versions (Solid, Vite, Pixi, Tone, mediabunny, etc.) live in **`package.json`** — cite that file instead of duplicating pins here.

## Stack (do not assume React / Next)

- **UI:** SolidJS (`*.tsx`), progressive port: some surfaces still mount from the imperative **`App`** class while Solid owns a subtree under `#solid-root` (see `src/main.tsx` comments).
- **Orchestration:** `createApp()` builds the store, constructs **`App`** (`src/app.ts`), `await app.init()`, returns `{ ctx, app }` for `AppCtx` + subsystem handles (`src/createApp.ts`).
- **State:** `src/store/` (`state.ts` store factory, `AppCtx.ts`, `watch.ts`, `eventSignal.ts`).
- **Graphics / theory / MIDI helpers:** Pixi.js + `pixi-filters`, **tonal**, `@tonejs/midi`, `@tonejs/piano`, **tone** (synth / scheduling).
- **MP4 export:** **`VideoEncoder` / `AudioEncoder` (WebCodecs) + Mediabunny** (`mediabunny`); offline audio via **`OfflineAudioRenderer`** (`src/export/VideoExporter.ts` header comment). Heavy export modules are **dynamic-imported** from `App` so they stay out of the initial bundle (`src/app.ts` comment near `VideoExporter`).

## Where things live (verified paths)

| Area                                             | Starting points                                                                                             |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| Entry                                            | `index.html` → `src/main.tsx`                                                                               |
| Bootstrap                                        | `src/createApp.ts`, `src/app.ts` (orchestrator), `src/AppRoot.tsx`                                          |
| Store                                            | `src/store/`                                                                                                |
| Clock / MIDI parse / persistence / service types | `src/core/` (`core/clock/MasterClock.ts`, `core/midi/parser.ts`, `core/persistence.ts`, `core/services.ts`) |
| Audio                                            | `src/audio/` (`AudioEngine.ts`, `SynthEngine.ts`, `Metronome.ts`, `OfflineAudioRenderer.ts`, …)             |
| MIDI I/O & recording                             | `src/midi/`                                                                                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aayushdutt/midee](https://github.com/aayushdutt/midee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
