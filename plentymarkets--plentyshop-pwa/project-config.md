---
trigger: always_on
description: This file provides guidance for LLM coding tools when working with this repository. Run `npm run llm:link all` to link it to provider-specific files (e.g. `AGENTS.md`, `GEMINI.md`).
---

# CLAUDE.md

This file provides guidance for LLM coding tools when working with this repository. Run `npm run llm:link all` to link it to provider-specific files (e.g. `AGENTS.md`, `GEMINI.md`).

## Overview

PlentyONE Shop PWA is a Turborepo monorepo providing a headless e-commerce frontend built with Nuxt 4, Vue 3, TypeScript, TailwindCSS, and Storefront UI 2. It connects to PlentyONE backend via the Alokai SDK and middleware.

**Workspaces:**

- `apps/web/` — Nuxt 4 PWA frontend (srcDir: `app/`)
- `apps/server/` — Alokai Middleware (Express.js), reads env from `apps/web/.env`
- `packages/shop-cli/` — PlopJS-based code generator

## Commands

```bash
# Dev (both frontend + middleware concurrently)
npm run dev

# Build / Start
npm run build && npm run start

# Unit tests (Vitest)
npm run test
npm run test:watch
npm run test:coverage

# E2E tests (Cypress) — requires API_ENDPOINT in apps/web/.env; each spins up `npm run start` first
npm run test:cypress                # paypal + dhl + feature + checkout + editor specs
npm run test:cypress-dev            # Open Cypress UI
npm run test:cypress-smoke-shop     # Shop smoke suite only
npm run test:cypress-smoke-editor   # Editor smoke suite only
npm run test:cypress-quarantine     # Flaky/quarantined specs

# Lint / Typecheck / Format
npm run lint
npm run lint:fix
npm run typecheck      # vue-tsc --noEmit (also runs as a build dependency)
npm run format         # prettier --check
npm run format:fix

# Clean
npm run clean          # Clear build artifacts
npm run clean:hard     # Full reset (node_modules, .nuxt, .turbo, dist)

# Code generation
npm run generate:component
npm run generate:composable
```

To run a single unit test file: `npm run test -- --run path/to/file.spec.ts`

Node version is specified in `.nvmrc`.

## Architecture

### Frontend (`apps/web/app/`)

- **`components/`** — 120+ Vue components. Each lives in its own folder: `ComponentName/ComponentName.vue`, `ComponentName/types.ts`, `ComponentName/index.ts`, `ComponentName/__tests__/ComponentName.spec.ts`
  - `ui/` — Generic StorefrontUI block components (shop base components)
  - `editor/` — CMS/editor base components
  - `blocks/` — CMS-style content blocks (dynamically loaded)
  - `settings/` — Admin configuration panels
- **`composables/`** — All business logic (140+). Same folder structure as components. State via `useState()` for SSR safety.
- **`pages/`** — File-based routing
- **`configuration/`** — i18n, `app.config.ts`, `feature-flags.config.ts`, `settings.config.ts`, `tailwind.config.ts`, `block-layout.config.ts`, `security.config.ts`
- **`utils/blocks/blocks-imports.ts`** — Dynamic block loader (maps block name → async import) via `getBlockLoader()`
- **`utils/settings-groups-imports/`** — Auto-discovers admin settings panels
- **`utils/triggers-imports/`** — Auto-discovers toolbar trigger components
- **`lang/`** — 26 locale JSON files (`en.json`, `de.json`, …)

### State Management

No global store. All state lives in composables via Nuxt's `useState()` (SSR-safe).

- **Parent ↔ child:** props and events.
- **Cross-component / cross-module:** composables, or `usePlentyEvent` from `@plentymarkets/shop-core`.

### i18n

26 languages in `app/lang/`. Use `t()` for frontend strings — never the global `$t`/`$n`/`n` (see Enforced constraints). Admin components (in `settings/` and `blocks/**/*Form.vue`) always use English via `getEditorTranslation()`.

### Dynamic Blocks

CMS blocks are loaded lazily by name via `getBlockLoader()` from `utils/blocks/blocks-imports.ts`. Any new block component must be registered there.

## Code Style

- **No magic string literals in code.** String values with semantic meaning that appear in comparisons or are referenced from multiple places (block names, event names, route keys, status enums, etc.) must live as exported constants in a dedicated module — not hardcoded inline. Re-use the constant everywhere the value is referenced.
  ```ts
  // ❌  if (block.name === 'UtilityBar') { ... }
  // ❌  if (block.name === 'Header') { ... }
  // ✅
  import { HEADER_BLOCK_NAME, UTILITY_BAR_BLOCK_NAME } from '~/utils/blocks/block-names';
  if (block.name === UTILITY_BAR_BLOCK_NAME) { ... }
  if (block.name === HEADER_BLOCK_NAME) { ... }
  ```
  Block-name constants live in `apps/web/app/utils/blocks/block-names.ts`. Add to that file when introducing a new block-name reference. For other categories (event names, etc.), add a similarly-scoped constants module rather than scattering literals.
- **Always brace conditionals and loops**, even one-liners. Format the braces on their own lines.
  ```ts
  // ❌  if (props.close) return;
  // ❌  while (el && el.tagName !== 'HEADER') el = el.parentElement;
  // ✅
  if (props.close) {
    return;
  }
  while (el && el.tagName !== 'HEADER') {
    el = el.parentElement;
  }
  ```
- **Don't over-annotate TypeScript.** If type inference produces the same result and the lint passes without an annotation, omit it. Only add annotations when they're load-bearing (the inferred type is wrong, too wide, or the lint fails without them).

  ```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plentymarkets/plentyshop-pwa](https://github.com/plentymarkets/plentyshop-pwa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
