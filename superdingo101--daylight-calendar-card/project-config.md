---
trigger: always_on
description: This repository contains Daylight Calendar Card, a Home Assistant custom dashboard calendar card.
---

## Project overview

This repository contains Daylight Calendar Card, a Home Assistant custom dashboard calendar card.

The project was formerly named Skylight Calendar Card. The public name is now Daylight Calendar Card, but some filenames and compatibility aliases intentionally still use `skylight`.

## Core rules

* Keep changes small, focused, and directly related to the prompt.
* Do not perform broad rewrites, formatting sweeps, dependency upgrades, or architectural refactors unless explicitly requested.
* Preserve `skylight-calendar-card.js` as the shipped HACS/manual-install artifact.
* Preserve both `daylight-calendar-card` and legacy `skylight-calendar-card` custom element compatibility.
* Reuse existing helpers, patterns, tests, and fixtures before adding new abstractions.
* Do not change defaults or visual behavior unless the prompt asks for it.
* Treat visual/layout changes as potentially breaking for dashboard users.

## Repository structure

* `src/` is the authored source of truth.
* `src/skylight-calendar-card.js` is the Rollup entry point and main custom element source.
* The root `skylight-calendar-card.js` file is the generated shipped artifact for HACS/manual installs.
* `rollup.config.mjs` builds `src/skylight-calendar-card.js` into the root `skylight-calendar-card.js` artifact.
* The generated root `skylight-calendar-card.js` artifact must remain committed.
* Do not hand-edit the root `skylight-calendar-card.js` file as source-of-truth. Make source changes in `src/`, then run `npm run build` and commit the regenerated artifact when it changes.
* `skylight-calendar-card.test.js` contains Node tests using `node:test`.
* `playwright/visual.spec.js` contains visual/browser behavior tests.
* `docs/` contains the Mintlify documentation site.
* `hacs.json` controls HACS metadata.

## Module boundaries

The broad modularization refactor is complete through the post-Phase-33 cleanup checkpoint. Current modules are organized around these boundaries. Future modules may be added, but new extractions should be tied to real feature or bug work and should preserve the same separation principles: keep card-instance orchestration in the main custom element, and keep extracted modules explicit, focused, and independent from card instance state.

* `src/skylight-calendar-card.js`: Rollup entry point and main custom element. It intentionally owns custom element registration and lifecycle; config orchestration; preference persistence; Home Assistant `hass` setter behavior; capability checks; final event/weather refresh decisions; final render timing; view composition; renderer callback wiring; DOM reads/writes; ResizeObserver behavior; compact-height measurement; scroll restoration; modal behavior; event listeners; service behavior; and Daylight/legacy Skylight compatibility wrappers.
* `src/version.js`: card version lookup helper. Do not update release/version behavior unless explicitly preparing a release.
* `src/translations.js`: translation data.
* `src/constants.js`: shared static constants.
* `src/defaults.js`: default config values, option lists, aliases, and stub config creation.
* `src/config/config-normalizers.js`: config normalization helpers for modes, colors, opacities, hidden calendars, and related public option values.
* `src/utils/date-utils.js`: date parsing, local date formatting, range chunking, and ISO week helpers.
* `src/utils/normalization-utils.js`: normalization helpers for enums, booleans, dashboard paths, and entity maps.
* `src/utils/string-utils.js`: string and HTML-attribute escaping helpers.
* `src/utils/color-utils.js`: color parsing, named-color handling, alpha blending, contrast, and color map normalization helpers.
* `src/ha/ha-state-helpers.js`: Home Assistant entity state display helpers, render signatures, person labels/pictures, and header weather display data.
* `src/events/event-normalizer.js`: Home Assistant calendar event normalization and combined-event data shaping.
* `src/events/event-display.js`: non-rendering event display decisions and display metadata.
* `src/events/event-fetcher.js`: calendar fetch/cache helpers, range coverage checks, stable signatures, merge/sort helpers, and WebSocket fetch orchestration helpers.
* `src/events/event-form.js`: event form validation, recurrence helpers, and create/update data normalization.
* `src/events/event-service.js`: Home Assistant calendar event service and WebSocket payload helpers.
* `src/rules/condition-matcher.js`: condition and value matching helpers.
* `src/rules/style-rules.js`: style rule normalization and matching helpers.
* `src/badges/day-badges.js`: day badge normalization, matching, template resolution, and display data helpers.
* `src/weather/weather-utils.js`: weather formatting, icon, temperature, and forecast utility helpers.
* `src/weather/weather-service.js`: weather entity discovery and Home Assistant weather service payload helpers.
* `src/weather/weather-controller.js`: weather forecast controller helper for forecast cache/refresh state. The main card still owns final weather refresh decisions and render timing.
* `src/editor/daylight-calendar-card-editor.js`: Daylight Calendar Card editor custom element registration and editor implementation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superdingo101/daylight-calendar-card](https://github.com/superdingo101/daylight-calendar-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
