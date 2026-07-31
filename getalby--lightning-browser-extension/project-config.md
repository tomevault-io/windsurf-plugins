---
trigger: always_on
description: This file provides guidance for AI coding agents working on the Alby Lightning Browser Extension repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the Alby Lightning Browser Extension repository.

## Project Overview

Alby Lightning Browser Extension — a WebExtension (Chrome MV3, Firefox/Opera MV2) that brings Bitcoin Lightning Network payments and Nostr signing to the browser. It implements the **WebLN**, **Nostr (NIP-07)**, **WebBTC**, and **Liquid** provider APIs by injecting them into web pages and routing requests through a background script that talks to a configurable Lightning backend (LND, LNbits, LNC, LNDhub, NWC, Alby account, Eclair, Galoy, plus native-companion variants for Tor).

## Tech Stack

- **Build:** Webpack 5 + SWC, separate builds per target browser (`dist/<env>/<browser>`)
- **Frontend (UI surfaces):** React 19, React Router v6, TypeScript (strict), Tailwind CSS 3, i18next, Zustand, axios
- **Storage:** `browser.storage` (settings/state) + Dexie/IndexedDB (allowances, payments, permissions, blocklist) via `db.ts`
- **Crypto:** `@noble/*`, `@scure/*`, `bitcoinjs-lib`, `liquidjs-lib`, `nostr-tools`, `crypto-js` (legacy)
- **Testing:** Jest (unit, via `@swc/jest`). The Playwright e2e suite under `tests/e2e/` is currently broken and unmaintained — rely on Jest only.
- **Manifest:** Single `src/manifest.json` with `__chrome__` / `__firefox__` / `__opera__` prefixes resolved by `wext-manifest-loader` at build time

## Common Commands

### Development

```bash
yarn install
yarn dev:chrome     # watch build → dist/development/chrome
yarn dev:firefox    # watch build → dist/development/firefox
yarn dev:opera
```

Load `dist/development/<browser>` as an unpacked / temporary extension. See `doc/SETUP.md` for browser-specific load instructions and `doc/SETUP_ANDROID.md` for Firefox Android.

### Build / Package

```bash
yarn build:chrome              # single-browser production build
yarn build                     # chrome + firefox + opera
yarn package                   # builds all + zips into dist/production via create-packages.sh
```

`manifest.json` ships with version `0.0.0`; the webpack loader rewrites it from `package.json` at build time.

### Lint, Format, Type-check

```bash
yarn lint            # eslint + tsc --noEmit + prettier --write (the canonical pre-commit check)
yarn lint:js         # eslint src --max-warnings 0
yarn lint:js:fix
yarn tsc:compile     # tsc --noEmit
yarn format          # prettier --check
yarn format:fix
```

ESLint, Prettier, and `tsc --noEmit` are enforced on staged files via Husky + lint-staged. Commit messages are validated by commitlint (Conventional Commits).

### Testing

```bash
yarn test:unit                                   # all jest tests
yarn test:unit path/to/file.test.ts              # single file
yarn test:unit -t "fragment of test name"        # by test name
yarn test:coverage
```

The Playwright `yarn test:e2e` / `yarn test` scripts still exist in `package.json` but the e2e suite is broken and not maintained — do not rely on them. Add new tests as Jest unit tests (colocated `*.test.ts` next to source, or under `tests/unit/`).

## Architecture

### Three execution contexts

The extension code is split across three runtime contexts that **cannot share modules or memory directly** — they communicate only via `browser.runtime.sendMessage` and `window.postMessage`.

1. **Background script** (`src/extension/background-script/`) — long-lived (MV2) / service-worker-like (MV3) singleton. Owns the unlocked wallet state, IndexedDB, the connector instance, and is the **only** place that touches the Lightning backend or private keys.
2. **Content scripts** (`src/extension/content-script/*.js`) — injected into every page. They inject the corresponding inpage script (`inpage-script/`) into the page's main world and proxy messages between page ↔ background. There is one content script per provider (`webln.js`, `alby.js`, `nostr.js`, `liquid.js`, `webbtc.js`) plus `onstart.ts` and `context-menu.ts`.
3. **UI surfaces** (`src/app/router/`) — separate React apps bundled as different webpack entries:
   - `popup` — the toolbar popup
   - `prompt` — modal window opened by the background script for user confirmation (payment, signMessage, permission, etc.)
   - `options` — full-page settings
   - `welcome` — onboarding flow

Each surface lives under `src/app/router/<Name>/index.tsx` and is referenced from `webpack.config.js` `entry:` and from `static/views/*.html`.

### Provider APIs (injected into web pages)

Inpage providers live in `src/extension/providers/<api>/index.ts`. Each method calls `this.execute(actionName, args)` which `postMessage`s to its content-script bridge; the bridge forwards to `router.ts` and the response comes back the same way (see [postMessage.ts](src/extension/providers/postMessage.ts) and [providerBase.ts](src/extension/providers/providerBase.ts)). All methods require `enable()` to have resolved first; `providerBase` enforces this. Public docs: <https://guides.getalby.com/developer-guide/developer-guide/alby-browser-extension-apis>.

**`window.webln`** — [providers/webln/index.ts](src/extension/providers/webln/index.ts) (WebLN spec)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getAlby/lightning-browser-extension](https://github.com/getAlby/lightning-browser-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
