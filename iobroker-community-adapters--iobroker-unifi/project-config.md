---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`iobroker.unifi` is an ioBroker adapter that polls a **UniFi Network controller** (classic controller or UniFi OS, e.g. UDM-Pro) via [node-unifi](https://github.com/jens-maus/node-unifi) and mirrors sites, clients, devices, WLANs, networks, health, vouchers, DPI statistics, gateway traffic and alarms into ioBroker states. A few states are writable and control the controller (WLAN on/off, PoE, LED, restart, block/reconnect clients, create vouchers).

TypeScript (CommonJS output). Sources live in `src/`, the published/runnable code is the compiled `build/` (`package.json` `main` is `build/main.js`). `build/` is gitignored — always run the build before starting the adapter or the tests.

## Commands

```bash
npm run build                             # tasks.ts: clean, tsc -p tsconfig.build.json -> build/, regenerate jsonConfig options
npm run 1-backend                         # only compile
npm run 2-jsonConfig                      # only regenerate the state filter options in admin/jsonConfig.json
npm run watch                             # tsc in watch mode
npm run check                             # type check only (tsconfig.json, noEmit)
npm run lint                              # eslint (@iobroker/eslint-config, flat config)
npx eslint -c eslint.config.mjs --fix src tasks.ts   # autofix + prettier formatting

npm run test:package                      # validates package.json / io-package.json / admin JSON (fast, no build needed)
npm run test:unit                         # mocha test/*.test.js against build/ (needs npm run build first)
npm run test:integration                  # starts a real js-controller + adapter instance
npx mocha test/refresh.test.js --exit --grep "backoff"   # single test
npm run release-patch                     # @alcalzone/release-script, moves README changelog into io-package news
```

There is deliberately **no `prepare` script** — `npm ci`/`npm install` does not build. Because `build/` is neither committed nor built on install, `common.nogit` is `true` in `io-package.json`: the adapter can only be installed from npm, not from GitHub. The integration test requires that **no** js-controller is running on the machine, otherwise it aborts with "JS-Controller is already running!".

## Architecture

### Layout

| Path | Content |
| --- | --- |
| `src/main.ts` | the whole adapter: one `Unifi extends utils.Adapter` class |
| `src/lib/jsonLogic.ts` | the custom json-logic operations used by the object definitions, `applyRule()` |
| `src/lib/objectDefinitions.ts` | loads `admin/lib/objects_*.json`, collects state IDs, builds the jsonConfig options |
| `src/lib/dpiNames.ts` | names of the DPI categories and applications (generated from the old `json_logic.js`) |
| `src/lib/types.ts` | object definitions, UniFi data, filters |
| `src/lib/adapter-config.d.ts` | augments `ioBroker.AdapterConfig` |
| `src/types/node-unifi.d.ts` | typings for node-unifi (only the methods used) |
| `admin/lib/objects_*.json` | **the object definitions** — which objects/states exist and how their values are computed |
| `admin/jsonConfig.json` | the configuration dialog |
| `tasks.ts` | build steps (run by `tsx`) |

`src/lib/adapter-config.d.ts` is hand-maintained and must be kept in sync with `native` in `io-package.json` **and** with `admin/jsonConfig.json`. `test/config.test.js` checks that jsonConfig and `native` have the same keys.

### Object definitions: `admin/lib/objects_*.json`

The states are not created in code. Each `objects_<category>.json` is a tree of definitions:

- `_id`, `type`, `common`, `native`, `value` — fixed parts, or the same keys under `logic` as a **json-logic rule** evaluated against the controller data. A plain string rule is short for `{ "var": "<string>" }`.
- `logic.has` + `logic.has_key` — children; `has_key` names the property of the data that holds their data (`_self` = the data itself). Arrays are iterated.
- `applyJsonLogic()` walks the tree, builds each object, writes it with `extendObjectAsync` **only when it changed** (`ownObjects` cache), converts the value to `common.type` and writes the state only when the value changed.

The custom operations (`cleanupForUseAsId`, `timestampToDateTime`, `translateAppCodeToName`, …) are registered on import of `src/lib/jsonLogic.ts`. The files are read from `join(__dirname, '..', '..', 'admin', 'lib')`, i.e. relative to `build/lib` — `admin/lib` must stay in the npm package (`files`).

### Refresh loop

`updateUnifiData()` → `performUpdate()`:

1. `getController('default')` logs in once and caches the `Controller` per site in `controllers`. node-unifi's own re-login is a no-op for an initialised controller, so an expired session is detected by `isAuthenticationError()` (401 or `api.err.LoginRequired`, **not** 403 = missing permission) and handled by `resetControllers()` + one retry — only if cached sessions were used.
2. `fetchSites()`, then for every site the fetch methods in `FETCH_METHODS` whose `update.*` flag is set. A failing site or endpoint is logged and skipped (reported in `info.lastError`), it does not abort the refresh.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iobroker-community-adapters/ioBroker.unifi](https://github.com/iobroker-community-adapters/ioBroker.unifi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
