---
trigger: always_on
description: This repository contains the frontend and integration code for `hass_datapoints`, a Home Assistant custom integration focused on:
---

# AGENTS.md

## Purpose

This repository contains the frontend and integration code for `hass_datapoints`, a Home Assistant custom integration focused on:

- interactive history charting
- annotation / datapoint creation and browsing
- target-row driven chart configuration
- comparison date windows
- reusable UI primitives for Home Assistant-flavoured Lit components

This file is the working project guide for contributors and coding agents. Use it as the source of truth for architecture, directory structure, component boundaries, testing, stories, and library placement.

---

## Core Tooling

- Use `pnpm` for Node-based workflows.
- Use Vitest for unit/spec coverage.
- Use Storybook for component stories and interaction coverage.
- Prefer `rg` for search and `rg --files` for file discovery.

### Main verification commands

1. `pnpm build`
2. `pnpm test`
3. `pnpm vitest run <focused spec files>`
4. `pnpm sb:build`

When making scoped changes, run focused Vitest first, then `pnpm build`, then broader verification as needed.

---

## Repository Conventions

- Prefer placing utility scripts, pure helpers, and simple shared functions under `custom_components/hass_datapoints/src/lib/`.
- Never use shortcut `if` / `else` clauses or single-line conditional bodies.
- Always include explicit opening and closing braces for every conditional block, including `if`, `else if`, and `else`.
- Avoid duplicating types across component, story, and test files.
- Prefer direct file imports over barrel imports unless a compatibility barrel already exists for a deliberate boundary.
- Runtime imports generally use `@/…` aliases. Styles imports stay relative.

---

## Architecture Overview

The UI layer is intentionally split by responsibility:

- `src/atoms`
  Smallest reusable UI primitives.
  Examples: inputs, toggles, handles, simple display widgets.

- `src/molecules`
  Composed reusable units built from atoms.
  Examples: target rows, chart shell, comparison tabs, sidebar sections.

- `src/cards`
  Self-contained Lovelace / records-style feature cards that are not chart implementations.
  Examples: action, dev-tool, list, quick.

- `src/charts`
  Larger chart-oriented feature surfaces and chart cards.
  Includes the history chart/card stack plus sensor/history chart surfaces.

- `src/panels`
  Full page/panel composition.
  The primary page is `panels/datapoints/datapoints.js`.

- `src/components`
  Integration-specific larger shared pieces that do not fit cleanly as atoms/molecules/cards.
  Current example: annotation dialog controller surface.

- `src/lib`
  Pure helpers, domain transforms, HA API utilities, state/session helpers, chart helpers, workers, i18n runtime, and general reusable logic.

### Layering guidance

- Atoms should not own page-level state.
- Molecules should compose atoms and emit clean events rather than reaching deep into child DOM.
- Cards should orchestrate feature-specific UI and behavior, but should still extract repeated subparts into atoms/molecules where reuse emerges.
- Panels own app/session state, URL state, and composition.
- `src/lib` should hold logic that can be tested without full DOM rendering whenever possible.

---

## Component vs Atom vs Molecule vs Card vs Page

### Atoms

Use an atom when:

- the UI is a single primitive or close to it
- the component is intended to be reused in many places
- the API can stay narrow and prop-driven
- the component should not know panel/page business rules

Examples:

- `atoms/form/radio-group`
- `atoms/interactive/range-handle`
- `atoms/display/feedback-banner`

### Molecules

Use a molecule when:

- you are composing two or more atoms into a reusable unit
- the unit has small internal interaction logic
- the unit still represents a reusable pattern rather than a page-specific feature shell

Examples:

- `molecules/target-row`
- `molecules/comparison-tab-rail`
- `molecules/sidebar-options`
- `molecules/panel-timeline`

### Cards

Use a card when:

- the unit is a user-facing feature surface with meaningful orchestration
- it owns feature-specific UI, config, or behavior
- it is not just a generic composition primitive

Examples:

- `cards/action`
- `cards/dev-tool`
- `cards/list`
- `cards/quick`
- `cards/history/history.ts`
- `cards/sensor/sensor.ts`

### Pages / Panels

Use a panel/page component when:

- the surface owns routing, URL/session state, layout, and feature composition
- it coordinates chart, sidebar, toolbar, list, dialogs, and persistent state

Primary example:

- `panels/datapoints/datapoints.js`

---

## Directory Structure Conventions

### General component directory structure

Preferred structure for reusable UI:

```text
src/<layer>/<component-name>/
├── <component-name>.ts
├── <component-name>.styles.ts
├── i18n/                        # optional but preferred for component-local translations
│   ├── fi.ts
│   └── fr.ts
├── types.ts                     # optional but preferred when types are shared
├── __tests__/
│   └── <component-name>.spec.ts
└── stories/
    └── <component-name>.stories.ts
```

### Localization structure

Frontend component translations now live next to the component they belong to in a local `i18n/` directory.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buggedcom/HASS-Data-Points](https://github.com/buggedcom/HASS-Data-Points) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
