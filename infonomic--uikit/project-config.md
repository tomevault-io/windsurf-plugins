---
trigger: always_on
description: This is a **framework-agnostic UI component library** using **CSS Modules** for styling (not Tailwind). The monorepo contains:
---

# UI Kit Copilot Instructions

## Project Overview

This is a **framework-agnostic UI component library** using **CSS Modules** for styling (not Tailwind). The monorepo contains:
- `packages/uikit` - Core component library (React + Astro)
- `apps/astro` - Astro demo app
- `apps/tanstack` - TanStack Router demo app

**Core Philosophy**: Components use CSS Modules to allow easy style overriding without `!important`. The library intentionally avoids Tailwind for component internals while remaining compatible with Tailwind-based consuming apps.

## Key Workflows

```bash
# Install and initial build
pnpm install && pnpm build

# Start all demo apps (parallel)
pnpm dev

# UIKit-specific commands
cd packages/uikit
pnpm storybook          # Component development
pnpm build              # Build for distribution
pnpm test               # Run Vitest tests
pnpm lint               # Biome linting
```

**Release Flow** (see `RELEASE-INSTRUCTIONS.md`):
1. `pnpm changeset` - Create changeset
2. GitHub Action auto-creates PR → merge → auto-publish to NPM
3. Manual: `pnpm version-packages` → `pnpm release:npm`

## Architecture Patterns

### Dual Export System
Components support **both React and Astro**:
- React: `src/react.ts` exports all components with `.js` extensions (TS requirement)
- Astro: `src/astro.js` exports select components as `.astro` files
- Consuming apps import via: `@infonomic/uikit/react` or `@infonomic/uikit/astro`

### Component Structure
Each component folder typically contains:
```
button/
├── @types/button.ts        # TypeScript interfaces
├── button.tsx              # React component
├── button.astro            # Astro variant (if supported)
├── button.module.css       # CSS Modules styling
├── button.stories.tsx      # Storybook stories
└── index.ts                # Barrel export
```

**Pattern**: Always import CSS modules as `styles` and use `cx()` from `classnames` for composition:
```tsx
import styles from './button.module.css'
import cx from 'classnames'

className={cx('button', intent, variant, styles.button, styles[variant], className)}
```

### CSS Architecture - CASCADE LAYERS (CRITICAL)

**CSS Cascade Layers are the foundation of style overridability.** CSS outside any layer automatically has higher specificity than CSS within layers - this is what allows consuming apps to easily override component styles without `!important`.

Every CSS module MUST include the layer preamble at the top:
```css
@layer infonomic-base, infonomic-utilities, infonomic-theme, infonomic-typography, infonomic-components;

@layer infonomic-components {
  /* component styles here */
}
```

**Layer Specificity Order** (lowest to highest):
1. `infonomic-base` - Reset/normalize styles, primitive tokens (colors, spacing)
2. `infonomic-functional` - **Semantic tokens** (intent, surface, field, text-role) — this is the semantic source of truth
3. `infonomic-utilities` - Utility classes
4. `infonomic-theme` - Document-level defaults and browser behaviour (autofill, scrollbars, element resets). Despite the name, semantic tokens live in `functional`, not here.
5. `infonomic-typography` - Typography styles
6. `infonomic-components` - Component styles
7. (unlayered) - Consumer app styles automatically win

**Why This Matters**:
- Enables per-component CSS bundling for tree-shaking (future: import only needed components)
- Consuming apps can override ANY style without `!important`
- Internal hierarchy lets functional tokens override base, components override functional, etc.

**Semantic Token System**:
- **Primitive tokens**: `src/styles/base/colors.css` - Base colors like `--primary-600`, `--red-500`
- **Semantic tokens**: `src/styles/functional/` - Intent, surface, field, and text-role tokens. This is the single source of truth for semantic styling. Each file (`colors.css`, `surfaces.css`, `typography.css`, `borders.css`) defines tokens in `:root`, `.dark`, and `.not-dark` scopes.

**Intent Token Naming**: `element-intent-emphasis-state` (e.g., `--fill-primary-strong-hover`)
  - `element`: `fill` (backgrounds), `text-on` (foreground on a fill), `text` (foreground with no fill context), `stroke` (borders), `ring` (focus rings), `gradient`
  - `intent`: `primary`, `secondary`, `noeffect`, `success`, `info`, `warning`, `danger`
  - `emphasis`: `strong`, `weak`, `outlined`, `text` (optional)
  - `state`: `hover`, `disabled` (optional)

**Canonical, not flattened**: Always use the full emphasis + state form. For the disabled state of a primary strong surface, use `--text-on-primary-strong-disabled`, not `--text-on-primary-disabled`. There is no `accent` intent family — the `--accent` variable is a raw brand palette token, not a surface token; prefer `--fill-primary-*` or `--surface-subtle-*` instead.

**Surface Token Naming**: `surface-type-state` (e.g., `--surface-item-hover`)
  - Used for: Dropdowns, selects, menus, tooltips, popovers, dialogs, command palettes
  - `surface-panel`: Container/viewport background (e.g., dropdown menu background)
  - `surface-panel-elevated`: Elevated panels with shadows (white in light, slightly lighter in dark)
  - `surface-panel-border`: Panel border color

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [infonomic/uikit](https://github.com/infonomic/uikit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
