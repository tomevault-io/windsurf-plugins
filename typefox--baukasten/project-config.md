---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Baukasten is a VSCode webview UI toolkit built with React 19, TypeScript, vanilla-extract, and designed to match VSCode's native look and feel. This is a monorepo managed with npm workspaces.

## Monorepo Structure

```
baukasten/
├── packages/
│   ├── baukasten/           # Main UI component library (baukasten)
│   ├── web-wrapper/         # VSCode theme wrapper for browser demos
│   ├── website/             # Documentation website
│   └── examples/
│       ├── web-example/     # Web application example
│       └── vscode/          # VSCode extension example (Log Viewer)
```

## Common Commands

### Development

```bash
# Install dependencies (run from root)
npm install

# Start developing the component library (with Vite)
npm run dev

# Start Storybook for component documentation
npm run storybook
# Runs at http://localhost:6006

# Run web example
npm run example:web
# Runs at http://localhost:3000

# Run VSCode extension example (then press F5 in VSCode)
npm run example:vscode
```

### Building

```bash
# Build all packages
npm run build

# Build specific packages
npm run build:baukasten
npm run build:web-wrapper

# Build Storybook documentation
npm run build-storybook
```

### Linting

```bash
# Lint the baukasten package (from packages/baukasten directory)
cd packages/baukasten
npm run lint
```

## Design System Architecture

Baukasten uses a **semantic design token system** that provides platform-agnostic theming. This is the most critical architectural pattern to understand.

### Token Files (Source of Truth)

All design tokens are defined in `packages/baukasten/src/styles/`:

- **colors.ts** - Semantic color tokens (primary, secondary, success, warning, danger, info, etc.)
- **spacing.ts** - Spacing scale, gaps, padding, component sizes
- **typography.ts** - Font sizes, weights, line heights, font families
- **effects.ts** - Border radius, shadows, transitions, opacity, z-index

### CSS Variables

Design tokens are exported as CSS custom properties (e.g., `--bk-color-primary`, `--bk-padding-md`, `--bk-font-size-md`) that map to VSCode theme variables by default but can be customized.

### Critical Rule: Use Semantic Tokens

**ALWAYS use semantic design tokens. NEVER hardcode values or use VSCode variables directly.**

```tsx
// ✅ CORRECT
background-color: var(--bk-color-primary);
padding: var(--bk-padding-md);
font-size: var(--bk-font-size-md);

// ❌ WRONG
background-color: #007acc;              // Never hardcode
padding: 8px 16px;                      // Never hardcode
background: var(--vscode-button-background);  // Too specific
```

### Component Structure

Each component follows this structure:

```
ComponentName/
├── ComponentName.tsx         # Implementation
├── ComponentName.css.ts      # vanilla-extract styles
├── ComponentName.stories.tsx # Storybook documentation
└── index.ts                  # Exports
```

**For components** (using vanilla-extract):

1. Use semantic tokens from the design system via CSS variables
2. Export TypeScript types for props
3. Include comprehensive Storybook stories
4. Use `recipe` API for variant-based styling
5. Use `style` API for base styles
6. Use `styleVariants` for simple variant maps

### Theming System

The project supports two environments:

**VSCode Webview** (native environment):

```tsx
import { GlobalStyles } from 'baukasten-ui/core';
// No wrapper needed - uses native VSCode theme
<GlobalStyles />;
```

**Web Application** (browser demos/Storybook):

```tsx
import { GlobalStyles } from 'baukasten-ui/core';
import { VSCodeThemeWrapper } from 'baukasten-ui-web-wrapper';
// Wrapper simulates VSCode theming
<>
    <GlobalStyles />
    <VSCodeThemeWrapper>...</VSCodeThemeWrapper>
</>;
```

## Storybook Guidelines

When creating Storybook stories, follow this exact order:

1. **Interactive** (FIRST - required) - Playground with all props exposed via controls
2. **Property Groups** - Variants, Sizes, States (grouped comparisons)
3. **Usage Examples** - WithIcons, WidthOptions, FormExamples (practical patterns)
4. **Showcase** (LAST - required) - Comprehensive overview with `layout: "fullscreen"`

All stories must:

- Use design tokens for spacing/styling
- Include `tags: ['autodocs']` in meta
- Have clear descriptions in `parameters.docs.description.story`
- Document all argTypes with descriptions and defaults

Reference: `packages/baukasten/src/components/Button/Button.stories.tsx`

## Available Components

Components are split across two entry points:

### Core Components (`baukasten-ui/core`)

**Form Controls:**

- **Button** - Versatile button with variants (primary, secondary, ghost, outline) and sizes
- **IconButton** - Square icon-only button matching Button height, for toolbar and compact actions
- **Input** - Text input with label and error state support
- **TextArea** - Multi-line text input
- **Select** - Dropdown selection component
- **Checkbox** - Checkbox with label support
- **Radio** - Radio button and RadioGroup components
- **Slider** - Range slider input

**Form Helpers:**

- **Label** - Form label component

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TypeFox/baukasten](https://github.com/TypeFox/baukasten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
