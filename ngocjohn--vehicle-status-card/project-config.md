---
trigger: always_on
description: **Vehicle Status Card** is a custom [Home Assistant](https://www.home-assistant.io/) Lovelace card built with TypeScript and [Lit](https://lit.dev/). It displays detailed status information about vehicles (indicators, range bars, images slideshow, a mini map, and interactive button cards) and is distributed as a single compiled JavaScript bundle via [HACS](https://hacs.xyz/).
---

# Copilot Instructions for Vehicle Status Card

## Project Overview

**Vehicle Status Card** is a custom [Home Assistant](https://www.home-assistant.io/) Lovelace card built with TypeScript and [Lit](https://lit.dev/). It displays detailed status information about vehicles (indicators, range bars, images slideshow, a mini map, and interactive button cards) and is distributed as a single compiled JavaScript bundle via [HACS](https://hacs.xyz/).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | TypeScript 5.x (strict mode, `experimentalDecorators`) |
| UI framework | Lit 3.x (LitElement, html\`\`, css\`\`) |
| Build | Rollup 4 + TypeScript plugin + PostCSS + Babel |
| Package manager | **pnpm 10** (do **not** use npm or yarn) |
| Lint | ESLint 9 (flat config, `eslint.config.mjs`) |
| Formatting | Prettier (`printWidth: 120`, single quotes, semicolons, 2-space indent) |
| Node version | **20** |

---

## Repository Layout

```
src/
  vehicle-status-card.ts   # Main custom element entry point (@customElement('vehicle-status-card'))
  components/              # Lit sub-components (vsc-* prefix)
    index.ts               # Re-exports all components
    shared/                # Shared/reusable components
  editor/                  # Visual editor for HA Lovelace panel
    editor.ts              # VehicleStatusCardEditor element
    components/            # Editor panel sub-components
  types/
    config/                # TypeScript interfaces for all config shapes
      card/                # Per-section config types (button-card.ts, indicators.ts, layout.ts, …)
      card-struct/         # superstruct validation schemas
    section.ts             # SECTION enum (INDICATORS, RANGE_INFO, IMAGES, …)
    config-area.ts         # ConfigArea enum (mirrors sections for the editor)
  constants/
    const.ts               # CARD_NAME, CARD_VERSION, COMPONENT enum
  utils/
    base-element.ts        # BaseElement – all card elements extend this (extends LitElement)
    store.ts               # Store – shared config/state passed down to all sub-elements
    editor/                # Editor-only helpers (migrate-*, reorder-*, clean-config, …)
    lovelace/              # HA Lovelace helpers (tap-action, create-card-element, …)
  ha/                      # Home Assistant type stubs and common utilities
  css/                     # Shared CSS (imported via PostCSS + postcss-lit)
  events/                  # Custom DOM events used between card and editor

build/                     # Production bundle output (vehicle-status-card.js)
dist/                      # Dev bundle output (watch mode)
scripts/                   # Maintenance scripts (update-bug-template.js, …)
```

---

## Development Commands

```bash
# Install dependencies (always use pnpm)
pnpm install

# Lint only (ESLint on src/**/*.ts)
pnpm run lint

# Build for production (lint + rollup → build/vehicle-status-card.js)
pnpm run build

# Watch / dev server (rollup → dist/vehicle-status-card.js, served on :8235)
pnpm run start

# Debug watch mode (sets DEBUG=true, enables verbose debugger output)
pnpm run debug

# Full clean then rebuild
pnpm run clean:build && pnpm run build
```

> **No test suite exists** – validation is done via lint + successful build only.  
> CI (`ci.yml`) runs `pnpm run lint` and `pnpm run build` on every push/PR that touches `src/**`.

---

## Code Style & Conventions

### General
- All source lives under `src/`. Never modify files under `build/` or `dist/` – they are generated.
- TypeScript strict mode is enabled; avoid `any` where possible even though `noImplicitAny` is off.
- Unused imports are a **warning** (eslint-plugin-unused-imports). Remove them.
- Imports must be sorted alphabetically (eslint-plugin-perfectionist `sort-imports`). Run `pnpm run lint` to catch violations.

### Naming
- Custom elements use the `vsc-` prefix (e.g., `vsc-buttons-group`, `vsc-mini-map`).
- The main card element is `vehicle-status-card` (constant `CARD_NAME`).
- Components are named `VscButtonsGroup`, `VscMiniMap`, etc. – Pascal-case dropping the prefix.
- Editor components follow the same naming scheme with `Editor` or `Panel` suffix.

### Component patterns
- All card-side elements extend `BaseElement` (which extends `LitElement`).
- Editor elements extend `BaseEditor`.
- Shared state is passed via the `Store` class (not context/redux), handed down as `_store`.
- Use `@property({ attribute: false })` for `hass` and `_store` properties.
- CSS is written in `.css` files under `src/css/` and imported via PostCSS + `postcss-lit` (outputs typed `CSSResult`).

### Config types
- All configuration shapes are defined as TypeScript interfaces under `src/types/config/card/`.
- `superstruct` schemas live in `src/types/config/card-struct/` and mirror the interfaces.
- `VehicleStatusCardConfig` (in `src/types/config/card/card-config.ts`) is the root config type.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngocjohn/vehicle-status-card](https://github.com/ngocjohn/vehicle-status-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
