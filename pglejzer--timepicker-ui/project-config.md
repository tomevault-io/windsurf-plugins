---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Coding Standards & Conventions

Read @.maister/docs/INDEX.md before starting any task. It indexes the project's coding standards and conventions:
- Coding standards organized by domain (frontend, backend, testing, etc.)
- Project vision, tech stack, and architecture decisions

Follow standards in `.maister/docs/standards/` when writing code — they represent team decisions. If standards conflict with the task, ask the user.

### Standards Evolution

When you notice recurring patterns, fixes, or conventions during implementation that aren't yet captured in standards — suggest adding them. Examples:
- A bug fix reveals a pattern that should be standardized (e.g., "always validate X before Y")
- PR review feedback identifies a convention the team wants enforced
- The same type of fix is needed across multiple files
- A new library/pattern is adopted that should be documented

When this happens, briefly suggest the standard to the user. If approved, invoke `/maister:standards-update` with the identified pattern.

## Maister Workflows

This project uses the maister plugin for structured development workflows. When any `/maister:*` command is invoked, execute it via the Skill tool immediately — do not skip workflows for "straightforward" tasks. The user chose the workflow intentionally; complexity assessment is the workflow's job.

## What This Is

`timepicker-ui` — a zero-dependency, framework-agnostic time picker library (v4.4.0). Supports analog clock, scroll wheel, and compact-wheel UI modes. SSR-safe. Published to npm as `timepicker-ui`. Three optional plugins (range, timezone, wheel) are tree-shaken out unless imported. 12 built-in CSS themes.

## Repository Layout

```
app/                    ← ALL source, build, and test work happens here
  src/
    timepicker/         ← main class: TimepickerUI, CoreState, Managers, Lifecycle
    managers/           ← 8 built-in managers (see Architecture below)
      clock/            ← ClockManager internals: engine/, controller/, renderer/, handlers/
      config/           ← ConfigManager internals: DisabledTimeHandler, InputValueHandler, MobileViewHandler
      events/           ← EventManager internals: ButtonHandlers, InputHandlers, KeyboardHandlers, ModalHandlers
      plugins/          ← plugin managers: range/, timezone/, wheel/
    core/               ← PluginRegistry (plugin system)
    plugins/            ← plugin definitions (RangePlugin, TimezonePlugin, WheelPlugin)
    utils/              ← EventEmitter, accessibility/, config/, debounce/, input/, template/, time/, validation/
    types/              ← TypeScript interfaces (.d.ts files): options, types, ITimepickerUI, images
    constants/          ← timings.ts (animation/delay constants)
    styles/             ← SCSS: index.scss, main.scss, variables.scss, _mixins.scss, themes/
  tests/
    unit/               ← mirrors src/ structure (80+ test files)
    __mocks__/          ← CSS and SVG mock stubs for Jest
  tsup.config.ts        ← JS build config (ESM + CJS → ../dist/)
  rollup.config.js      ← CSS themes + .d.ts declarations (→ ../dist/css/, ../dist/)
  webpack.config.js     ← dev server only
  jest.config.ts        ← test config
  tsconfig.json         ← dev tsconfig
  tsconfig.prod.json    ← production tsconfig (used by tsup)
  tsconfig.test.json    ← test tsconfig
dist/                   ← generated output, never edit
docs-app/               ← Next.js 16 documentation site (deployed on Vercel)
```

## Commands

**All commands run from the `app/` directory.**

```bash
cd app

# Dev server
npm run start                           # webpack dev server

# Lint
npm run eslint                          # ESLint only (src/**/*.ts)
npm run lint                            # lint-staged + pretty-quick (for staged files)

# Build
npm run build:prod                      # eslint → tsup → rollup (full production build)
npm run build:tsup                      # JS only (ESM + CJS)
npm run build:rollup                    # CSS themes + .d.ts declarations only

# Test
npm test                                # all tests (Jest)
npm run test:unit                       # tests/unit/** only
npm run test:watch                      # watch mode
npm run test:coverage                   # with coverage
npm run test:verbose                    # verbose output
npm run test:ci                         # CI mode (--ci --coverage --maxWorkers=2)

# Single test file
npx jest tests/unit/managers/ClockManager.test.ts

# Single test by name
npx jest -t "should rotate clock hand"

# Tests matching a pattern
npx jest --testPathPattern="clock"
```

### Docs app (separate project)

```bash
cd docs-app
npm run dev                             # Next.js dev server
npm run build                           # Next.js build
```

## Architecture

### Composition pattern — TimepickerUI

`TimepickerUI` (the public API class) does not contain business logic itself. It composes three internal objects:

```
TimepickerUI
├── CoreState      — state bag (degrees, options, DOM refs, flags, getters/setters)
├── Managers       — creates and owns all 8 managers + plugin managers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pglejzer/timepicker-ui](https://github.com/pglejzer/timepicker-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
