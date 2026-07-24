---
trigger: always_on
description: This guide helps AI agents understand the Frontile project structure and development workflow.
---

# Frontile Development Guide for AI Agents

This guide helps AI agents understand the Frontile project structure and development workflow.

## Project Overview

Frontile is a modern, accessible component library for Ember.js built with:

- **Ember Octane** with Glimmer components
- **TypeScript** with Glint for type-safe templates
- **Tailwind CSS** with Tailwind Variants for styling
- **GTS/GJS** template tag format (prefer `.gts` over `.gjs`)
- **pnpm workspaces** for monorepo management

## Project Structure

**Important — source is consolidated.** All component source now lives in the single
`packages/frontile` package, grouped by category under `packages/frontile/src/components/`.
The old per-feature packages (`buttons`, `collections`, `forms`, `overlays`, `notifications`,
`status`, `utilities`) are now thin **deprecation/re-export wrappers** — their `src/` contains
only `index.ts` + `template-registry.ts` and re-exports from `frontile`. **Do not add or edit
component source in those wrapper packages**; work in `packages/frontile/src/`.

```
frontile/
├── packages/
│   ├── frontile/                    # PRIMARY package — all component source lives here
│   │   └── src/
│   │       ├── components/          # Source grouped by category:
│   │       │   ├── buttons/         #   Button, ButtonGroup, Chip, CloseButton, ToggleButton
│   │       │   ├── collections/     #   Table, Listbox, Dropdown
│   │       │   ├── forms/           #   Input, Select, Checkbox, Radio, Switch, Textarea
│   │       │   ├── overlays/        #   Modal, Drawer, Popover, Overlay, Portal
│   │       │   ├── notifications/   #   NotificationCard, NotificationsContainer
│   │       │   ├── status/          #   ProgressBar
│   │       │   └── utilities/       #   Avatar, Collapsible, Divider, Spinner, VisuallyHidden
│   │       ├── modifiers/  services/  utils/
│   │       └── buttons.ts, collections.ts, …  # category barrel entry points
│   ├── theme/                       # Styling system (Tailwind Variants + semantic colors)
│   ├── core/                        # Shared low-level primitives
│   ├── tailwindcss-plugin-helpers/  # Helpers for the Tailwind plugin
│   ├── changeset-form/              # Form components integrated with ember-changeset
│   ├── forms-legacy/                # Legacy form components (still maintained)
│   └── buttons/ collections/ forms/ overlays/ notifications/ status/ utilities/
│                                    # Deprecation wrappers re-exporting from `frontile`
├── test-app/             # Test application (integration/unit tests for all components)
├── site/                 # Documentation site (frontile.dev), built with Docfy
└── docs/                 # Markdown docs (theming, migrations) rendered by the site

```

**Component docs are co-located:** each component has a sibling `.md` file next to its `.gts`
(e.g. `packages/frontile/src/components/buttons/button.gts` + `button.md`). See the
Documentation section below — these `.md` files are rendered as **live demos** by Docfy.

## Development Workflow

### Running Tests

**All tests:**

```bash
cd test-app && pnpm ember test
```

**Filter specific tests:**

```bash
cd test-app && pnpm ember test --filter="table"
```

**From root (alternative):**

```bash
pnpm test
```

### Building Packages

**Build the main package (most component work):**

```bash
pnpm --filter frontile build
```

**Build the theme package (after any style/color change):**

```bash
pnpm --filter @frontile/theme build
```

**Build all packages:**

```bash
pnpm build
```

**Important:** When working on a component, build `frontile` before running tests. If you
modify styles or colors in `@frontile/theme`, build `@frontile/theme` too (the theme feeds
generated Tailwind classes/CSS variables consumed everywhere). The legacy per-feature
packages (`collections`, `buttons`, …) are wrappers — you rarely build them directly.

### Linting

**Check linting:**

```bash
pnpm lint:js
```

**Auto-fix linting:**

```bash
pnpm lint:js --fix
```

### Type Checking

**Check types for a specific package:**

```bash
pnpm --filter frontile lint:types
pnpm --filter @frontile/theme lint:types
```

**Check types for test-app:**

```bash
cd test-app && pnpm lint:types
```

**Always run type checking and linting before committing.**

### Running Development Server

```bash
pnpm start  # Starts test-app dev server
```

## Code Style Guidelines

### Component Format

- **Prefer `.gts` over `.gjs`** for all components
- For template-only components, use TypeScript and import `TOC` (Template Only Component)
- Use Glimmer components with `<template>` tags
- Follow TypeScript strict mode conventions

Example template-only component:

```typescript
import type { TOC } from '@ember/component/template-only';

export interface Signature {
  Args: {
    title: string;
  };
}

const MyComponent: TOC<Signature> = <template>
  <div>{{@title}}</div>
</template>;

export default MyComponent;
```

### Helpers and Utilities

**Important:** When working with template helpers in `.gts` files:

- Import helpers explicitly from `@ember/helper` (e.g., `hash`, `array`, `fn`)
- **`eq` is NOT available** from `@ember/helper` - use custom helper or inline comparison

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josemarluedke/frontile](https://github.com/josemarluedke/frontile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
