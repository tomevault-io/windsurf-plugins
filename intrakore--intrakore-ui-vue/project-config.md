---
trigger: always_on
description: frappe-ui library development standards and best practices
---


# Project Name: frappe-ui

**frappe-ui** is a Vue 3 component library and utilities package designed for
rapid development of modern Frappe-based web applications.

## Overview

frappe-ui provides a broad set of Vue 3 components, utility layers,
Frappe-oriented integrations, docs tooling, and styling primitives used across
multiple Frappe products.

This file defines the **general development guidance for the repository**.

## How to use v1-release docs

The docs in `v1-release/` are important **planning and direction documents**.
They should **inform** implementation decisions in areas they cover, but they
should not replace this file as the general repo-wide guidance.

Use them primarily when working on:

- core component stabilization
- component API consistency
- selection/menu family APIs
- deprecation and migration decisions
- TextEditor narrowing and stabilization
- release-facing docs and changelog updates

Especially relevant files:

- `v1-release/plan.md`
- `v1-release/04-components-audit.md`
- `v1-release/08-selection-and-menu-api-spec.md`
- `v1-release/changelog.md`

If you are working outside those topics, follow the normal repo guidance in this
file.

## Tech Stack & Architecture

- **Frontend Framework**: Vue 3 with Composition API and `<script setup>` syntax
- **Language**: TypeScript-first authoring for public APIs and modern components
- **Build Tool**: Vite
- **Styling**: TailwindCSS v3 with semantic color tokens
- **UI Primitives**: Reka UI / accessible headless primitives
- **Rich Text**: TipTap v3
- **Date Utilities**: dayjs
- **Testing**: Vitest
- **Documentation**: VitePress

## Project Structure

```bash
frappe-ui/
├── src/
│   ├── components/               # Main library source code
│   ├── data-fetching/            # Vue 3 composables for Frappe API access
│   ├── resources/                # Older resource APIs kept for compatibility
│   ├── mocks/                    # Mock service worker handlers and test helpers
│   ├── composables/              # Reusable Vue composables
│   ├── directives/               # Vue directives
│   ├── utils/                    # Shared utilities
│   └── index.ts                  # Main export surface
├── docs/                         # VitePress docs site
├── icons/                        # Custom icon components
├── tailwind/                     # Tailwind preset/plugin/colors
├── vite/                         # Vite plugins and helpers
├── frappe/                       # Frappe-specific components and utilities
└── v1-release/                   # Planning docs for v1 stabilization work
```

## Development Standards

### General engineering principles

- Treat frappe-ui as a **reusable library**, not an app codebase
- Prefer stable, high-signal public APIs over clever internal abstractions
- Optimize for maintainability, consistency, and migration safety
- Keep component responsibilities narrow and composable
- Avoid growing components into do-everything abstractions
- Prefer small, requirement-driven changes over speculative rewrites
- When a behavior is public, preserve backwards compatibility unless the task
  explicitly involves a planned breaking change or deprecation path

## Component Authoring Guidelines

### Component structure and API design

When creating or modernizing a component, prefer this structure:

```bash
src/components/ComponentName/
├── ComponentName.vue
├── types.ts
├── utils.ts           # optional
├── stories/           # optional but preferred for public components
└── index.ts
```

For small legacy components that do not yet match this structure, prefer moving
incrementally toward it instead of forcing a large rewrite unless the task calls
for one.

### TypeScript requirements

For public component APIs, use consistent naming:

- `ComponentNameProps`
- `ComponentNameEmits`
- `ComponentNameSlots`
- `ComponentNameExposed`
- `ComponentNameSize`, `ComponentNameVariant`, `ComponentNameTheme`, etc.

These names matter because public types and JSDoc are used to generate
component documentation metadata.

### JSDoc expectations

Every public prop, emit, and slot should have a JSDoc description.

Use JSDoc for:

- generated docs
- IntelliSense quality
- clear API contracts for consumers

### Vue authoring conventions

Prefer:

- `<script setup lang="ts">`
- `defineProps<Props>()`
- `defineEmits<Emits>()`
- `defineSlots<Slots>()`
- `defineExpose<Exposed>()` when truly needed
- `withDefaults()` for sensible defaults
- `computed` instead of `watch` when deriving state
- small, focused composables for reusable logic

Avoid:

- Options API for new component work unless there is a compelling migration reason
- runtime prop validation when TypeScript is enough
- broad watchers when a narrower reactive dependency would do
- imperative DOM access unless necessary

### Public API design rules

Prefer:

- narrow and predictable component boundaries
- props and slots as the default customization mechanism
- `v-model` / `modelValue` for the primary value state
- `v-model:open` for visibility state where relevant
- semantic event names like `change`, `open`, `close`, `submit`
- slots over render functions for normal customization
- limited, requirement-driven escape hatches

Avoid:

- giant multi-mode components with too many flags

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intrakore/Intrakore_UI.vue](https://github.com/intrakore/Intrakore_UI.vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
