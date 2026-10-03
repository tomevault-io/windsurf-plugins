---
trigger: always_on
description: This file applies to the entire `ytzero-enhance` repository. Keep it current when a product decision, integration contract, build workflow, or non-obvious browser workaround changes.
---

# Repository working guide

This file applies to the entire `ytzero-enhance` repository. Keep it current when a product decision, integration contract, build workflow, or non-obvious browser workaround changes.

## Project purpose

YT Zero Enhance is the browser-extension companion for the self-hosted YT Zero application. It supports Chromium browsers, Firefox, and Safari on macOS/iOS/iPadOS from one TypeScript codebase.

The sibling `../ytzero` repository owns the application-side bridge. Its relevant contract is documented in `../ytzero/docs/browser-extension-integration.md`, and its configuration element is emitted by `../ytzero/ui/src/App.tsx`. Do not modify the sibling repository unless the user explicitly puts it in scope.

## Source layout

- `src/background.ts` — extension background messaging, paired-instance registry, redirects, captures, and page/player coordination.
- `src/content.ts` — top-page bridge plus embedded player controls and shortcuts. It runs in all matching frames.
- `src/instances.ts` — embedded configuration parsing, instance URL inference, matching, and settings URLs.
- `src/contract.ts` — validated, versioned application/player bridge contract and security boundaries.
- `src/core.ts` — browser-independent URL, settings, filename, timestamp, and screenshot geometry helpers.
- `src/popup.ts` and `src/options.ts` — extension UI behavior.
- `static/` — popup/options HTML and CSS; `static/icons/icon.svg` is the icon source of truth.
- `_locales/{en,pl,de}` — browser locale catalogs. Every user-facing key must exist in all three locales.
- `manifests/` — per-browser Manifest V3 inputs.
- `scripts/build.ts` — builds all browser targets, generates raster icons, syncs Safari resources, validates manifests, and creates ZIP packages.
- `PRIVACY.md` — authoritative English privacy policy linked from the README and store listings.
- `safari/YT Zero Enhance/` — checked-in containing-app/Xcode wrapper.
- `tests/` — Bun unit tests and browser UI preview fixtures.

## YT Zero pairing contract

- A signed-in YT Zero page inserts `#ytzero-enhance-configuration` as JSON inside `<body>` on every application route, not only watch pages.
- Pairing must work from any authenticated YT Zero page, including `/`, settings, history, plugin/future routes, and watch pages.
- In `src/popup.ts`, read and validate the embedded configuration before accepting an instance. The application manifest URL is used as the preferred base-URL hint.
- `inferInstanceUrl()` must retain support for an installation prefix such as `/apps/ytzero`, including when a reverse proxy exposes the app below a path.
- Do not trust configuration solely because an element has the expected ID. Always validate format, version, bridge version, events, and field shapes through the contract helpers.
- Pairing an instance requests optional host access only after the user initiates the action. Keep host permissions and origin matching as narrow as the browser APIs allow.
- Firefox loses the user-action status after an awaited promise. Inspect and validate the active pairing candidate while the popup opens, then invoke `permissions.request()` synchronously from the Connect button handler before awaiting any pairing work.
- Static content scripts run only on the required embedded-player hosts. Register `content.js` dynamically for the exact origins of paired instances and unregister it when the last instance on an origin is removed; never restore a static `<all_urls>` match.
- Do not add the `tabs` permission: `activeTab`, paired-origin access and the fixed player-host permissions cover the current tab operations.
- Multiple instances are supported. Each page uses its matching instance/profile; only supported video-link redirects use the default instance.

## Embedded player interaction decisions

- Treat validated `context.video.contentType` (`default`, `short`, or `livestream`) as the source of truth for player presentation. Missing or unknown values retain the legacy `/shorts` and native-live fallbacks; valid context changes apply in place without reloading the iframe.
- Shorts controls start hidden and must only reveal after pointer movement or a click/tap on the player. Autoplay, pause events, initial setup, and context changes while scrolling must not reveal them.

- Player presentation has three modes: standard, live, and shorts. The background derives shorts mode from the paired top-page `/shorts` route and verifies live mode through a narrowly scoped `MAIN`-world player probe; the iframe also keeps DOM/media fallbacks.
- Live mode shows a live-edge control, uses the seekable DVR window for progress, disables frame stepping and 2× hold, and keeps playback at 1×. Shorts mode uses compact circular controls, a lighter timeline, hides the expanded volume slider, elapsed-time label, and PiP button, and routes Up/Down from a focused iframe to the parent short-form navigator.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pelski/ytzero-enhance](https://github.com/Pelski/ytzero-enhance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
