---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Homebridge plugin (`homebridge-eveatmo`) that exposes a Netatmo Weatherstation and/or Indoor Air Quality monitor as HomeKit accessories, mimicking the look and feel of Elgato Eve Room/Weather devices (custom Eve characteristics, Eve history via `fakegato-history`) rather than using stock HomeKit sensor services.

Plain JavaScript (CommonJS, `require`/`module.exports`), not TypeScript, despite `typescript`/`typescript-eslint` being devDependencies — those are only used to power the ESLint flat config's type-aware rules.

## Commands

```bash
npm run lint     # eslint . --max-warnings=0
npm run fix      # eslint . --max-warnings=0 --fix
npm test         # node --test
```

There is no build step. Tests use Node's built-in `node:test` runner — see `test/` (mirrors the source layout: `device/`, `accessory/`, `lib/`, plus `index.test.js` at the root) and `test-helpers/` (shared bootstrap; deliberately kept outside any directory named `test`, since `node --test`'s default discovery recursively treats every `.js` file under a `test/`-named directory as a test file). Two distinct mocking styles are used depending on the layer:

- `device/`/`accessory/` tests (which exercise `service/*.js` indirectly, through the accessories that build them) run against the real `@homebridge/hap-nodejs` (no hand-built HAP stub), with `fakegato-history` neutralized via a `require.cache` substitution (`test-helpers/stub-fakegato-history.js`) since it touches real disk/timers even at construction time.
- `lib/netatmo-api.js` tests stub `axios` the same way (`test-helpers/stub-axios.js`) rather than hitting real HTTP — that module's public methods and its module-level (not per-instance!) auth state make it necessary to keep each auth scenario in its own test file, one process per file.

`index.js`'s `EveatmoPlatform` class isn't exported directly — capture it via a fake `homebridge.registerPlatform` (`test-helpers/load-platform.js`). Note: `EveatmoPlatform.accessories()` is not covered end-to-end, because the device instances it builds internally start an un-`unref()`'d poll `setInterval` with no way to reach in and clear it from outside — calling it in a test hangs the process. CI (`.github/workflows/build.yml`) runs `npm install` + `npm run lint` + `npm test` on Node 20.x/22.x/24.x for pushes to `master`/`develop` and PRs.

To manually exercise the plugin without a real Netatmo account, set `"mockapi": "eveatmo"` in the platform config — this routes API calls to `lib/netatmo-api-mock.js`, which reads fixture JSON from `mockapi_calls/*-<mockapi value>.json` (falls back to `*-default.json` if the named fixture is missing).

## Architecture

Everything is wired together through `homebridge`'s `hap` object, which is only available once Homebridge calls the plugin's export function. Because of this, almost every file in this repo follows the same shape:

```js
module.exports = function (pHomebridge) {
  if (pHomebridge && !homebridge) {
    homebridge = pHomebridge;
    // require() sibling modules here, passing `homebridge` through
  }
  class Something extends SomeHapClass { ... }
  return Something;
};
```

Modules are required and instantiated lazily inside constructors/builders (not at top-level) specifically so `homebridge.hap.*` classes exist before they're subclassed. When adding a new accessory/service, follow this same factory-function pattern rather than requiring `homebridge.hap` at module load time.

### Layers (top to bottom)

1. **`index.js`** — registers the `eveatmo` platform. `EveatmoPlatform` validates config (`auth` block, `grant_type`), constructs the Netatmo API client (real or mock), and in `accessories()` fans out to one "device type" per enabled feature (`weatherstation`, `airquality`) via `async.parallel`, retrying once after 30s on failure.

2. **`lib/netatmo-api.js`** — hand-maintained, promise-free (callback + EventEmitter) client for the Netatmo REST API (axios under the hood). Handles both `refresh_token` and deprecated `password` OAuth grants; persists refreshed tokens to `<homebridge-storage>/netatmo-token.json` and proactively refreshes 5 minutes before expiry. This file is a maintained fork of the abandoned `netatmo` npm package — treat API surface changes here as intentional, not accidental duplication. `lib/netatmo-api-mock.js` is a drop-in replacement with the same method names, backed by static fixtures.

3. **`device/*-device.js`** (`WeatherstationDeviceType`, `AirQualityDeviceType`) — extend `lib/netatmo-device.js`'s `NetatmoDevice` base class. Each owns a `NodeCache` (TTL from config, min 300s) for the raw Netatmo device map, a `setInterval` poll loop that pushes fresh data to all built accessories, and a `buildAccessory(deviceData)` factory that maps a Netatmo module `type` string (e.g. `NAMain`, `NAModule1`) to an accessory class. Whitelist/blacklist filtering by device ID happens here in `buildAccessories()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skrollme/homebridge-eveatmo](https://github.com/skrollme/homebridge-eveatmo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
