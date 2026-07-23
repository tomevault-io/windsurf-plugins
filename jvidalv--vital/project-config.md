---
trigger: always_on
description: This document helps AI assistants like Claude understand and work effectively with the Vital project.
---

# Claude AI Assistant Guide

This document helps AI assistants like Claude understand and work effectively with the Vital project.

## Project Overview

**Vital** is a modern React starter template built with the latest versions of Vite, React, TypeScript, and Tailwind CSS. It follows Atomic Design principles and includes comprehensive code quality tools.

## Tech Stack

### Core Framework
- **Vite 7.2.4** - Next-generation frontend build tool
- **React 19.2.0** - Latest React with improved performance
- **TypeScript 5.9.3** - Strict type checking enabled
- **Node.js**: >=18 required
- **Package Manager**: Yarn >=1.22.5

### Styling
- **Tailwind CSS v4.1.17** - Latest version with CSS-based configuration
- **@tailwindcss/forms 0.5.10** - Form styling plugin
- **PostCSS 8.5.6** with Autoprefixer 10.4.22

### Code Quality
- **ESLint 9.39.1** - Flat config format (ESLint 9+)
- **typescript-eslint 8.47.0** - TypeScript linting
- **Prettier 3.6.2** - Code formatting
- **lint-staged 16.2.7** - Pre-commit linting
- **Commitlint 20.0.0** - Conventional commits enforcement

### Additional Libraries
- **@heroicons/react 2.2.0** - Beautiful SVG icons

## Project Structure

```
vital/
├── public/
│   ├── favicon.svg
│   └── manifest.webmanifest
├── src/
│   ├── app/
│   │   └── app.tsx                    # Main App component
│   ├── components/                     # Atomic Design architecture
│   │   ├── atoms/                      # Basic building blocks
│   │   │   ├── button/
│   │   │   │   ├── button.tsx
│   │   │   │   └── index.ts
│   │   │   └── logos/
│   │   ├── molecules/                  # Simple component groups
│   │   │   └── copy-button/
│   │   ├── organisms/                  # Complex components
│   │   │   └── card/
│   │   └── templates/                  # Page layouts (empty)
│   ├── hooks/                          # Custom React hooks
│   ├── index.css                       # Global styles + Tailwind
│   ├── main.tsx                        # Entry point
│   └── vite-env.d.ts
├── commitlint.config.js
├── eslint.config.js                    # ESLint flat config
├── index.html                          # HTML entry
├── lint-staged.config.js
├── postcss.config.js
├── tailwind.config.js
├── tsconfig.json                       # Root TS config
├── tsconfig.app.json                   # App TS config
├── tsconfig.node.json                  # Node/Vite TS config
└── vite.config.ts
```

## Path Aliases

**IMPORTANT**: This project uses path aliases configured in both `vite.config.ts` and `tsconfig.app.json`.

### Available Aliases
```typescript
// Instead of: import Button from '../../components/atoms/button'
// Use:
import Button from "components/atoms/button";
import App from "app/app";
import { useCustomHook } from "hooks/useCustomHook";
```

**Configured paths:**
- `app/*` → `src/app/*`
- `components/*` → `src/components/*`
- `hooks/*` → `src/hooks/*`

## Available Scripts

```bash
yarn dev      # Start dev server on http://localhost:3000
yarn build    # TypeScript check + Vite build → dist/
yarn lint     # Run ESLint with auto-fix
yarn preview  # Preview production build locally
```

## TypeScript Configuration

### Key Settings (tsconfig.app.json)
- **Target**: ES2020
- **Module**: ESNext with Bundler resolution
- **JSX**: react-jsx (new transform, no React import needed)
- **Strict mode**: Enabled ✅
- **Additional strictness**:
  - `noUnusedLocals: true`
  - `noUnusedParameters: true`
  - `noFallthroughCasesInSwitch: true`
  - `noUncheckedSideEffectImports: true`

### Important
- Unused variables/parameters will cause compilation errors
- All imports must be properly typed
- Use `allowImportingTsExtensions: true` (Vite handles this)

## Tailwind CSS v4 Setup

**CRITICAL**: This project uses **Tailwind CSS v4**, which has a **CSS-first configuration** approach.

### Configuration (Pure CSS - NO tailwind.config.js)

Tailwind v4 does NOT use `tailwind.config.js`. All configuration is done in CSS using special directives.

### CSS Configuration (src/index.css)
```css
@import "tailwindcss" source(".");
@plugin "@tailwindcss/forms";

@layer base {
  /* Custom base styles */
}
```

**Key directives:**
- `@import "tailwindcss" source(".")` - Imports Tailwind and sets content path (relative to CSS file)
- `@plugin "@tailwindcss/forms"` - Loads official plugins
- `@theme { }` - Define custom design tokens (colors, spacing, fonts, etc.)
- `@utility { }` - Create custom utility classes
- `@source` - Additional content paths or safelisting

**Key differences from v3:**
- **NO tailwind.config.js** - All configuration in CSS
- No separate `@tailwind base/components/utilities` directives
- Use `@import "tailwindcss"` instead
- Content paths specified with `source()` function (relative to CSS file) or `@source` directive
- Plugins loaded via `@plugin` directive, not JS config
- Theme customization via `@theme` directive with CSS variables
- Requires `@tailwindcss/postcss` package for PostCSS integration

## Atomic Design Architecture

This project follows the Atomic Design methodology:

### Hierarchy
1. **Atoms** - Basic building blocks (Button, Logos)
2. **Molecules** - Simple component groups (CopyButton)
3. **Organisms** - Complex components (Card)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvidalv/vital](https://github.com/jvidalv/vital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
