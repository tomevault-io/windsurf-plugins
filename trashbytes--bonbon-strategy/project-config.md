---
trigger: always_on
description: Guidance for AI agents working in this repository. This file is derived from the current source; keep it in sync when you change behavior or structure.
---

# AGENTS.md — Bonbon Strategy

Guidance for AI agents working in this repository. This file is derived from the current source; keep it in sync when you change behavior or structure.

## What this is

A Home Assistant **Lovelace dashboard strategy**. It is not a card — HA calls the strategy's `generate(userConfig, hass)` and it returns a full dashboard (`{ views: [...] }`) built from Bubble Cards. Distributed via HACS as a `plugin` (see `hacs.json`).

- Entry point / custom element: `bonbon-strategy.js` → `customElements.define('ll-strategy-bonbon-strategy', BonbonStrategy)`.
- Users reference it as `type: custom:bonbon-strategy`.
- Requires `custom:bubble-card`. `mini-graph-card` and `card-mod` are optional but change behavior when present.

## File map

All modules are plain ES modules loaded in the browser. There is **no build step and no bundler**.

| File | Role |
| --- | --- |
| `bonbon-strategy.js` | `BonbonStrategy.generate()` — orchestrates everything. Uses top-level `await` and `import.meta.url`. |
| `bonbon-strategy-config.js` | Exports `defaultConfig` (the `views → sections → cards` tree). |
| `bonbon-strategy-utils.js` | Color math (`getColorsFromColor`, `getAreaColors`, `getWeatherIcon`), `mergeDeep`, `normalizeSectionColumn`, `applySectionColumns`, `upgradeConfig`. |
| `bonbon-strategy-builders.js` | `createBuildersApi(panelUrl, config)` → `createButtonCard`, `createSeparatorCard`, `createSubButton`, `createGrid`, `isTogglableEntity`, `hasBinaryState`. |
| `bonbon-strategy-styles.js` | `createStylesApi(panelUrl, config)` → `css`, `observeDarkMode`, `cssVariable`, `cssValue`, `getVariables`, `getStyles`. |
| `bonbon-strategy-entities.js` | `createEntityApi(ctx)` → entity prep, selector resolution, sorting, area/floor scoping. |
| `README.md` | User-facing documentation. **Source of truth for config options** — update it whenever you add or change a user-facing option. |
| `hacs.json` | HACS metadata (`filename: bonbon-strategy.js`). |
| `.github/workflows/validate.yaml` | CI: `hacs/action` validation (daily + manual). No tests, no lint, no typecheck. |

Local-only / gitignored (do not assume they ship): `bonbon-strategy-loader.js` (dev cache-busting loader that appends `?hacstag=<timestamp>`), `workspace.js` (commented-out scratch), `assets/`, `ftp*`, `.history/`.

## Architecture / data flow

1. `generate(userConfig, hass)` is called. It stores a per-dashboard global namespace `window.bonbon[hass.panelUrl]`, exposing `resolveEntity` / `resolveEntities` (used by runtime `:hide()` styles).
2. Config = `upgradeConfig(mergeDeep(defaultConfig, userConfig))`.
3. `createEntityApi` is built from `hass.entities/states/devices/floors/areas`; `prepareEntities` flattens device → area → floor and merges device labels onto entities.
4. Area colors are computed and written as CSS variables (`--area-<id>-{light,medium,shade}-color`) for light and dark; `observeDarkMode` swaps variables on `<html>`.
5. `bonbon_home` is expanded: `bonbon_areas` is replaced by one section per floor (`bonbon_floor_<floor_id>`), each containing one area card per area.
6. `bonbon_area` is expanded into a **subview per area** (`bonbon_area_<area_id>`).
7. Every remaining view's sections are sorted by `order`, resolved into cards, prefixed with a separator card, wrapped in a `grid`, and converted from an object map to a `sections` array via `applySectionColumns`.
8. `applyGlobalStyles` recursively injects Bubble Card / card-mod styling, then returns `{ views }`. Errors are caught and returned as a Markdown "Error" view.

Note: `console.log(dashboard)` runs at the end of `generate` — useful in the browser console when debugging.

## Config model

`defaultConfig` shape: `{ styles, actions, views: { <viewKey>: { max_columns, subview?, disabled?, sections: { <sectionKey>: {...} } } } }`.

- Built-in views: `bonbon_home` (pushed first, title = dashboard title) and `bonbon_area` (templated per area, then deleted). Any other `views` key becomes a custom view with path `custom_<key>`.
- Sections are `disabled`-filtered and sorted by `order` (ascending, `Infinity` last).
- Section keys are meaningful: `bonbon_weather` and `bonbon_miscellaneous` have special handling in code, and `bonbon_area` sections use `area_id` scoping.
- Cards in a section may be **entity selector strings**, YAML card objects, or a mix. Special token `area.<attribute>` (e.g. `area.temperature_entity_id`) is replaced by the current area's attribute during expansion.
- `mergeDeep` deep-merges plain objects but **replaces arrays wholesale** (arrays are not treated as mergeable objects). This matters for `cards`, `inline_buttons`, etc.
- `upgradeConfig` maps legacy/renamed options onto the new shape (e.g. `show_weather_card`, `show_temperature`, `show_floor_lights_toggle`). When you rename or restructure a config option, add a back-compat mapping here.

## Entity selectors

Parsed in `resolveEntities` (`bonbon-strategy-entities.js`):

- Wildcards: `light.*`, `sensor.*battery`, `*temperature*` (converted to a regex over `entity_id`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trashbytes/bonbon-strategy](https://github.com/trashbytes/bonbon-strategy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
