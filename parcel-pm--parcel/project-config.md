---
trigger: always_on
description: | Purpose | Command | Notes |
---

# Parcel AI Agent Instructions

## Build, test, and lint commands

| Purpose | Command | Notes |
| --- | --- | --- |
| Build the shared extension bundle | `make extension` | Runs `make -C src`, formats source with Prettier, and writes generated assets to `src/dist/`. |
| Build the Chrome bundle | `make chrome` | Rebuilds `src/dist/` and syncs it into `chrome/`. |
| Build the Firefox bundle | `make firefox` | Rebuilds `src/dist/`, syncs it into `firefox/`, rewrites the manifest for Firefox, and switches module content scripts to the `.es6.js` shim. |
| Format source | `make prettier` | Formats `test/*.{js,json}` and `test/setup/*.js`, then runs `make -C src prettier`, which writes all `src/**/*.{js,json,less,css,html,xhtml}`. |
| Install dev dependencies | `make install-deps` | Runs `npm install --ignore-scripts --before 2026-06-10` to fetch dev-time dependencies. Use this instead of a bare `npm install` so the additional security arguments are respected. |
| Clean generated artifacts | `make clean` | Removes `src/dist/`, `chrome/`, `firefox/`, top-level `dist/`, and the generated `parcel-setup.sh`. |
| Run all tests | `make test` | Runs prettier/eslint syntax checks and shellcheck linting, then the full test suite with `node --test` across all `test/*.test.js` files. |
| Run individual test groups | `make test-native`, `make test-browser-mock`, `make test-modules`, `make test-application`, `make test-syntax`, `make test-setup` | Native-host integration tests; Chrome-API mock tests; shared-module unit tests; application tests; syntax tests; setup-script tests respectively. |
| Lint only | `make lint` | Runs ESLint semantic checks only. For lint with prettier checks (as run by CI), use `make test-syntax`. |
| List outstanding TODO comments | `make todo` | Prints all TODO comments found in `src`, `test`, and root `.md` files (excluding `AGENTS.md` itself), with author and date from git blame. |

### Test files

All files below live in `test/` and are run by `make test`. The `test/setup/*.test.js` suite is run separately by `make test-setup`.

| File | Covers |
| --- | --- |
| `chrome-api-mock.test.js` | The reusable Chrome-API mock itself. |
| `helpers.test.js` | Shared helpers: shadow-aware DOM lookups, TOTP, crypto utilities. |
| `native-host.test.js` | Native host end-to-end in isolated environments with mocked GPG. |
| `plaintext.test.js` | Parsing/expansion of decrypted entry data. |
| `schema.test.js` | Config schema validation, defaults, and warnings. |
| `selectors.test.js` | Field-selector registry, including `additionalSelectors`. |
| `targets.test.js` | Fill/extraction target mappings, including `additionalTargets`. |
| `shadow.test.js` | `main-world/shadow.js` attachShadow interception shim. |
| `webauthn.test.js` | WebAuthn encoding helpers and attestation-object builders. |
| `main-world-webauthn.test.js` | `main-world/webauthn.js` isolated-world installer and ceremony shim. |
| `agent.test.js` | Background agent: config validation, entry caching, port brokering. |
| `integration.test.js` | Content script: target detection and autofill behaviour. |
| `popup.test.js` | Popup UI: match listing, decrypted plaintext, fill relay. |
| `popup-context.test.js` | Popup per-origin/per-container history. |
| `popup-passkey.test.js` | Popup passkey-mode UI path. |
| `popup-warnings.test.js` | Popup security-warning display. |

Do not use `src/publicsuffix` as Parcel test guidance unless the task explicitly targets that vendored subtree.

## High-level architecture

- `src/` is the canonical source tree. `src/dist/`, `chrome/`, and `firefox/` are generated outputs; edit source files under `src/`, not generated bundles.
- The browser-side runtime is split into three main pieces:
  - `src/js/agent.js` is the MV3 background/service-worker coordinator. It owns native messaging, bootstraps the native host, validates config with `ConfigSchema`, caches entry lists, and brokers runtime ports.
  - `src/js/integration.js` is the content script injected into all frames at `document_start`. It detects fill targets, opens the inline/context popup, fills fields, and handles the broadcast "best target" autofill path.
  - `src/js/popup.js` is the toolbar/context popup UI. It requests matches and decrypted plaintext from the background worker, relays fill commands back into the active frame, and stores per-origin/per-container history in `chrome.storage.local`.
- Shared behavior lives in `src/js/helpers.js`, `src/js/plaintext.js`, `src/js/schema.js`, `src/js/selectors.js`, and `src/js/targets.js`. The intended config extension points are `additionalSelectors` and `additionalTargets`.
- Shadow DOM support is deliberate: `src/js/main-world/shadow.js` patches `attachShadow`, and cross-shadow lookups are expected to go through `Helpers.shadowSelector()` / `Helpers.shadowSelectorAll()`.
- The native side is split in two:
  - the repo-root `parcel-host` bootstrap host, which verifies signatures and loads the bundled script,
  - `src/parcel-host`, the signed host implementation that reads `~/.password-store`, filters entries against `.parcel.json`, and decrypts only paths that were previously whitelisted by `action_list`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parcel-pm/parcel](https://github.com/parcel-pm/parcel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
