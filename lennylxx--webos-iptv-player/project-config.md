---
trigger: always_on
description: IPTV player for LG webOS TVs. Vanilla TypeScript (no UI framework), bundled with
---

# AGENTS.md

IPTV player for LG webOS TVs. Vanilla TypeScript (no UI framework), bundled with
esbuild and packaged as a webOS `.ipk`. A separate bundled webOS JS service
(`bundled-service/`) provides LAN M3U uploads over Luna + HTTP. App id
`com.lennylxx.iptv`; targets webOS 4+.

## Commands

```bash
npm install                    # setup
npm run typecheck              # tsc --noEmit (TS strict mode — see TS strictness)
npm run service:smoke          # build + gate + real Node.js 0.12.2 service smoke
npm run service:smoke:matrix   # representative webOS 4-26 Node runtime matrix
npm run lint                   # stylelint + eslint webOS 4 / Chromium 53 browser-compat gate
npm run build                  # typecheck + esbuild bundle into dist/
npm run preview                # build + serve dist/ at http://localhost:3000 (desktop video via hls.js/mpegts.js)
npm test                       # vitest run (unit/integration)
npm run test:watch             # vitest watch
npm run test:e2e               # Playwright against the preview server (2 projects)
npm run test:all               # lint + unit + e2e
npm run screenshots            # regenerate README screenshots
./build.sh                     # package the IPK (needs ares-package from @webos-tools/cli)
./build.sh --install [device]  # build + ares-install + cold-restart on a TV
```

Run a single test by file or name:

```bash
npx vitest run src/parsers/m3u-parser.test.ts
npx vitest run -t "parses catchup-source"
```

There is **no Prettier/autoformatter**, but ESLint + stylelint are a real static
gate (`npm run lint`) alongside `tsc` strictness — see the webOS 4 compat gate under
Conventions. Run `npm run typecheck`, `npm run lint`, and the relevant tests before
considering a change done.

Before **every commit**, run both full suites against the final staged changes:

```bash
npm test
npm run test:e2e
```

Targeted UT/E2E runs are sufficient while iterating, but never replace these
pre-commit full-suite runs. Do not commit unless both pass after the final code
change.

## CI

`.github/workflows/build.yml` runs typecheck (app, `service`, **and** `benchmarks`),
`npm run lint` and the compatibility gate, `vitest run`, the esbuild bundle, and
packages the IPK. Pushes/PRs to `main` build; tagged `v*` pushes publish a GitHub
release with the `.ipk`.

After bundled-service changes, run `npm run service:smoke`. It downloads the
official Node.js 0.12.2 archive, verifies its published SHA-256, caches it under
the user cache directory, then parses and exercises the compiled service with
that exact runtime. CI runs `npm run service:smoke:matrix` across Node.js 0.12.2,
8.12.0, 12.21.0, 16.19.1, and 20.12.2, representing webOS 4 through 26. On
Apple Silicon, Node 16/20 use arm64 while older releases use x64 through Rosetta.

## Versioning

`package.json` `version` is the **single source of truth**. `esbuild.config.mjs`
syncs it into `appinfo.json` and the `__APP_VERSION__` build constant;
`scripts/sync-version.mjs` runs on `npm version`. **Never hand-edit the version in
`appinfo.json`.**

## Architecture

- **Entry `src/app.ts`** — the `App` class instantiates the top-level components, owns a
  `viewStack`, and routes remote-control input. `KeyHandler`
  (`src/navigation/key-handler.ts`) maps key codes (`CONFIG.KEYS`) to a small
  `Action` union (`up`/`down`/`select`/`red`/… in `src/types.ts`); `App.handleKey`
  dispatches to the active view's `handleAction`.
- **Views** are plain `<div>`s in `index.html` (`channels`, `player`, `epg`,
  `settings`, `loading`, plus the Xtream `movies`/`series` catalogs and `search`)
  toggled with `show`/`hide` from `src/utils/dom.ts`.
- **Sections & Xtream catalog** — a persistent `TabBar` (`src/components/tab-bar.ts`)
  docks the **Live / Guide / Movies / Series / Settings / Search** sections above the views and
  hosts the multi-account avatar switcher (`account-switcher.ts`); `App` owns section
  switching and a `viewStack`. Movies/Series (`movies.ts`, `series.ts`) share the
  browse/grid/detail machinery in `CatalogView` (`catalog-view.ts`) over the per-account
  `xtream-catalog` cache, with a **Continue Watching** resume rail. `search.ts` ranks
  channels, programs, movies, and series in one view; Movies/Series require an Xtream
  account, while Search remains available for channels and programs. M3U-only setups
  see Live/Guide/Settings/Search.
- **Components** (`src/components/`) own a DOM subtree and re-render through
  `morph()` (`src/utils/morph.ts`), a keyed in-place DOM reconciler fed by the
  `html` tagged template. Reused nodes keep their listeners, focus, and scroll — so
  list items carry a stable `data-key`. **Do not rebuild subtrees with
  `innerHTML =`**; build a `Safe` with `` html`…` `` and pass it to `morph`. Bind
  listeners once (delegated), not per render. `Player` delegates media loading and
  desktop hls.js/mpegts.js/Shaka access to `PlayerPipeline` (`player-pipeline.ts`) and
  audio/subtitle state to `PlayerTracks` (`player-tracks.ts`). A desktop MSE library
  owns its own tracks behind the `MseEngine` adapters in `src/components/mse/`
  (`isMseActive()`); on webOS everything plays natively, DASH included — see
  `docs/mpeg-dash.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lennylxx/webos-iptv-player](https://github.com/lennylxx/webos-iptv-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
