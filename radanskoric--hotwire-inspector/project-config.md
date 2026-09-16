---
trigger: always_on
description: This file contains critical context for any agent working on this project.
---

# Agents

This file contains critical context for any agent working on this project.

## Project Overview

This repository contains **Hotwire Inspector**, a cross-browser DevTools extension for inspecting Hotwire-related page structure, including Turbo Frames and Stimulus controllers.

Core tooling currently includes:

- WXT
- Vite
- Vitest
- Playwright
- ESLint
- Prettier
- Stylelint
- TypeScript (checkJs, JSDoc-only)

## Important Files and Areas

- `docs/PLAN.md`
  - the phased implementation plan and broader project notes from initial implementation
- `docs/decisions/`
  - key technical decisions and rationale for the project
  - check this folder before making architecture or tooling changes
- `entrypoints/content.js`
  - content script logic for scanning, element lookup, highlighting, and inspect support
  - also kicks off the tree watcher, page-lifecycle watcher, and Turbo event watcher on startup
- `entrypoints/devtools/`
  - DevTools page entrypoint that registers the Hotwire Inspector panel via WXT's `browser.devtools.panels.create`
- `entrypoints/panel/`
  - DevTools panel UI and interaction logic
  - tabs: Hotwire Elements (tree) and Turbo Events (live event timeline)
- `lib/tree-builder.js`
  - pure tree-building logic used by the panel
- `lib/controller-values.js`
  - pure Stimulus value logic; used by the injected page script (definition extraction), the content script (attribute resolution), and the panel (formatting)
- `lib/value-watcher.js`
  - MutationObserver-based Stimulus value watcher; used by the content script for live value streaming
- `lib/turbo-event-watcher.js`
  - capture-phase `document` listener for a curated set of Turbo DOM events; pushes each fire over the events port as a `TurboEventPush`
- `lib/port-reconnect.js`
  - port reconnection helper with exponential backoff; used by the content script for the events port
- `lib/panel/controllers/turbo-events-controller.js`
  - Stimulus controller owning the Turbo Events timeline (`lib/panel/controllers/turbo-events-controller.js`): cap, dropped-count notice, smart autoscroll, and the bridge subscription that drives `addEvent(name)`
- `lib/types.js`
  - shared JSDoc typedefs for the panel ↔ background ↔ content script ↔ inspected page messaging shapes (no runtime exports)
- `tests/unit/`
  - unit tests
- `tests/e2e/helpers.js`
  - shared Playwright helpers: Xvfb management, Chromium extension context, devtools frame discovery, content-script messaging
- `tests/e2e/content-script.spec.js`
  - E2E tests for the content-script pipeline (scanning, highlighting, inspect, parent-child relationships)
- `tests/e2e/panel.spec.js`
  - E2E tests for the panel UI rendering (tree nodes, badges, summary, empty/error states, refresh)
  - see the E2E Testing section below for how both files work

## Critical Project Expectations

- Do not mutate the target page DOM for tracking purposes.
- Preserve the current in-memory element identity approach unless there is a strong reason to change it. Always ask before making such changes.
- Prefer changes that keep the inspected page behavior as close to untouched runtime behavior as possible.
- Keep the extension cross-browser friendly.
- If you change central behavior in the content script or panel messaging, review the related E2E tests.

### Messaging architecture

The extension uses two messaging channels between the panel and the content script:

1. **One-shot relay** (panel → background → content script): the panel sends a `scan`, `highlight`, `unhighlight`, `inspect`, `watchValues`, or `unwatchValues` request via `browser.tabs.sendMessage` through the background relay. The content script responds via `browser.runtime.sendMessage`.
2. **Port-based event channel** (content script → background → panel): for live updates the content script opens a `ContentEventsPort` via `browser.runtime.connect` and pushes `ValuesChangedPush`, `TreeChangedPush`, or `TurboEventPush` messages.
   - `ValuesChangedPush` carries the current values of a watched controller.
   - `TreeChangedPush` is a lightweight signal that the inspected DOM may have changed and the panel should request a fresh scan. The background script also emits `TreeChangedPush` directly to the panel port when a tab finishes loading (`tabs.onUpdated` with `status === 'complete'`), so the tree refreshes after a reload or navigation. The content script listens to `pageshow` for back-forward-cache restores.
   - `TurboEventPush` carries `{ name, timestamp }` for each Turbo DOM event fired on the inspected page (see [Turbo Events capture](#turbo-events-capture) below).
     The background script routes every content-port message to the matching panel's `PanelEventsPort`. Ports provide `onDisconnect` lifecycle awareness for clean teardown.

### Turbo Events capture

- **What is captured**: a curated list of Turbo 8.x DOM events (`lib/turbo-event-watcher.js` exports `TURBO_EVENT_NAMES`). The list is the single source of truth — both the watcher and any tests that need to assert coverage import it from this module. Add a new event by adding a string to that array; nothing else needs to change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radanskoric/hotwire-inspector](https://github.com/radanskoric/hotwire-inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
