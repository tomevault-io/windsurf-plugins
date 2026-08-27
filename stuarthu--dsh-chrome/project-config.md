---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`dsh-chrome` is a browser companion for DeepSeek Harness (dsh): a Chrome MV3 extension whose side panel embeds the dsh web UI, plus three dsh host plugins that let the agent read the current page, capture HTTP traffic (via CDP), and drive the browser.

## Commands

There is no build step, no test suite, and no linter — the package ships plain JS source directly (`package.json` has no scripts).

- Install host plugins into dsh: `dsh plugin --profile web add dsh-chrome` (dsh hot-applies plugin rows; a browser refresh suffices unless an already-loaded plugin file was edited — then restart dsh web).
- Install/refresh the unpacked extension: `npx dsh-chrome install` (also `path`, `uninstall`) — loaded via `chrome://extensions` → Developer mode → Load unpacked. **This copies rather than symlinks**, so after editing anything under `extension/` you must re-run install *and* hit reload in `chrome://extensions`, or Chrome keeps running the old copy. This is the most common way to "fix" something and see no change.
- Release: update `CHANGELOG.md`, bump the version in **both** `package.json` and `extension/manifest.json` (they desync easily — the manifest was left a version behind once already), tag `vX.Y.Z` and push the tag — `.github/workflows/publish.yml` publishes to npm via OIDC trusted publishing. The tag must match `package.json`'s version or the workflow fails.
- `tools/*.cjs` are dev-only diagnostics over dsh's zstd-framed session logs, not shipped (`tools/` is absent from `package.json` `files`): `verify-intent.cjs <session-file>` and `dump-session.cjs <session-file> [seq|from-to]…`, both reading through `tools/session-log.cjs`.

### Verifying a change

With no test suite, "verified" means one of: `node --check` on the touched files; `node tools/verify-intent.cjs <session-file>` after any intent-gate change (it imports the production module, so it cannot drift from it); a throwaway `node -e` harness against `host/redact.js` or `host/intent-gate.js`, the two modules with no `@deepseek-ai/*` imports and therefore the only ones runnable standalone; and for anything under `extension/`, reinstalling and reloading it against a running `dsh web` (default `http://127.0.0.1:3080`). Say which you did — don't call an unexercised change verified.

## Architecture

Two halves talking over one WebSocket:

1. **Chrome extension** (`extension/`): `src/background.js` is the only place that touches Chrome APIs. It keeps a reconnecting WebSocket to `ws://<dsh>/dsh-agent/bridge`, executes `action` frames from dsh (get_page / list_tabs / navigate / click / open_tab / start_capture / stop_capture / capture_requests), captures the active tab's traffic via `chrome.debugger` (CDP), pushes debounced (~2 s) "current page" snapshots on tab switch / navigation / SPA route change (active tab only, deduped by URL + body length), and talks to the side panel over a `chrome.runtime` port. The side panel (`sidepanel.*`) is a thin iframe around the dsh web UI plus a status top bar (currently Chinese-only UI text). `injectable()` decides **from the tab URL, before trying** whether a page can take an injected script; those it rules out — non-`http(s)` (`chrome-extension://`, `chrome://`, `file://`) plus the Chrome Web Store — are read *and clicked* exclusively over the remote debugging endpoint `http://127.0.0.1:9222`. This is not a post-failure fallback: an ordinary `http(s)` page never takes the CDP path, a PDF served over https included.

2. **dsh host plugins** (`host/`), mounted by `cordis.patch.yml` when the package is added to a dsh profile — each row is addressed by id so users can override/disable it:
   - `bridge.js` (`dsh-chrome-bridge`): registers the `/dsh-agent/bridge` WS route on the dsh web server and exposes the `dshAgentBridge` service (`call(action, params, timeoutMs)`, `onPage`, `currentPage`, plus an `isConnected` that nothing currently calls). The other two plugins consume it.
   - `browser-tools.js` (`dsh-chrome-browser-tools`): registers the `browser_*` agent tools. Runs redaction (`redact.js`) over captured traffic before it reaches the model (config `redactCredentials`, default true).
   - `page-injector.js` (`dsh-chrome-page-injector`): subscribes to page pushes and injects a "current page" message into the most recently active session (the one that last received a real user message).

`host/` also holds two **non-plugin** modules, absent from both `cordis.patch.yml` and `package.json` `exports` because they're imported by relative path: `redact.js` and `intent-gate.js`. Both `tools/*.cjs` scripts reach the latter through `loadIntentGate()` in `tools/session-log.cjs` (a dynamic `import()` from CJS), which is why `intent-gate.js` must stay free of `@deepseek-ai/*` imports.

The wire protocol (JSON frames: `action`/`result`/`page`/`ping`/`pong`, plus `graph-changed` broadcast by the host to make the panel refresh) is documented in `docs/bridge-protocol.md`. If you change the protocol, update `extension/src/background.js`, `host/bridge.js`, and that doc together.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stuarthu/dsh-chrome](https://github.com/stuarthu/dsh-chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
