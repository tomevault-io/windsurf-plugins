---
trigger: always_on
description: **Last Updated:** September 30, 2025
---

# Claude's Learning Document - Chakra UI

**Last Updated:** September 30, 2025

## Project Overview

**Chakra UI** is a comprehensive React component system for building accessible,
high-quality web applications and design systems. It emphasizes speed,
accessibility, and works seamlessly with modern frameworks like Next.js
(including RSC support).

- **Repository:** https://github.com/chakra-ui/chakra-ui
- **Author:** Segun Adebayo
- **License:** MIT
- **Version:** 2.0.0
- **Package Manager:** pnpm 10.15.0
- **Node Version:** >=20.x

## Project Structure

### Root Directory

```
chakra-ui/
├── packages/           # Core packages
├── apps/              # Applications (www docs, mcp server)
├── sandbox/           # Testing environments
├── scripts/           # Build and utility scripts
├── .changeset/        # Changesets for versioning
└── package.json       # Root package configuration
```

### Key Packages

1. **`packages/react`** - Main React component library
   - Core system implementation
   - Styled-system utilities
   - Component recipes
   - Type definitions

2. **`packages/cli`** - CLI tools for Chakra UI
   - Type generation
   - Token management

3. **`packages/charts`** - Chart components

4. **`packages/panda-preset`** - Panda CSS preset for Chakra

### Apps

1. **`apps/www`** - Documentation website (https://chakra-ui.com)
2. **`apps/mcp`** - MCP server for AI integrations
3. **`apps/compositions`** - Component compositions examples

### Sandbox Environments

Multiple sandbox projects for testing in different environments:

- `next-app/`, `next-pages/` - Next.js testing
- `vite-ts/`, `vite-jsx/` - Vite testing
- `remix-ts/` - Remix testing
- `react-router/` - React Router testing
- `storybook-ts/` - Storybook testing
- `shadow-dom/` - Shadow DOM testing

## Styled System Architecture

The styled-system is the heart of Chakra UI's styling engine. Located in
`packages/react/src/styled-system/`:

### Core Files

1. **`system.ts`** - Main system creation and configuration
   - Creates the Chakra system instance
   - Integrates tokens, breakpoints, conditions, and utilities
   - Provides CSS-in-JS functionality

2. **`cva.ts`** (Class Variance Authority) - Recipe creation
   - Handles variant-based styling
   - Processes base styles, variants, and compound variants
   - **Recent Fix:** Now properly handles bracket syntax for responsive styles
     in recipes

3. **`sva.ts`** (Slot Variance Authority) - Multi-slot recipe creation
   - For components with multiple parts (slots)
   - Each slot can have its own variants

4. **`token-dictionary.ts`** - Token management
   - Token resolution and transformation
   - Semantic token handling
   - Color palette expansion

5. **`breakpoints.ts`** - Responsive breakpoint system
   - Converts breakpoint names to media queries
   - Handles array syntax for responsive values

6. **`conditions.ts`** - Conditional styling
   - Pseudo-selectors (:hover, :focus, etc.)
   - Data attributes
   - Media queries

7. **`utility.ts`** - Utility function management
   - Property transformations
   - CSS property mappings

8. **`calc.ts`** - CSS calc() utilities
   - Mathematical operations in CSS

### Style Resolution Flow

1. User defines styles (tokens, recipes, variants)
2. `normalize()` processes raw style objects
3. Breakpoint arrays `[value1, value2]` → responsive object
   `{base: value1, sm: value2}`
4. Tokens get resolved from token dictionary
5. Conditions get applied (media queries, pseudo-selectors)
6. CSS output is generated with proper layering

## Recent Work: Recipe Bracket Syntax Fix

### Problem

Array/bracket syntax for responsive styles wasn't working correctly in recipe
variants:

```typescript
const recipe = system.cva({
  variants: {
    variant: {
      primary: {
        color: ["red", "green"], // ❌ Wasn't converting to breakpoints
      },
    },
  },
})
```

### Solution

Enhanced the normalization process in the recipe system to handle bracket syntax
properly. The fix ensures that:

1. Base styles support bracket syntax
2. Variant styles support bracket syntax
3. Compound variants support bracket syntax
4. Mixed bracket and object syntax works correctly

### Modified Files

- `packages/react/src/styled-system/breakpoints.ts`
- `packages/react/src/styled-system/calc.ts`
- `packages/react/src/styled-system/conditions.ts`
- `packages/react/src/styled-system/cva.ts`
- `packages/react/src/styled-system/sva.ts`
- `packages/react/src/styled-system/system.ts`
- `packages/react/src/styled-system/token-dictionary.ts`
- `packages/react/src/styled-system/utility.ts`
- `packages/react/src/utils/entries.ts`

### Test Coverage

Created comprehensive tests in `packages/react/__tests__/recipe.test.ts`:

1. ✅ Bracket syntax in recipe variants (original issue)
2. ✅ Bracket syntax in base styles
3. ✅ Object syntax as control test
4. ✅ Multiple properties with bracket syntax
5. ✅ Multiple breakpoints (3+ values)
6. ✅ Compound variants with bracket syntax
7. ✅ Mixed bracket and object syntax
8. ✅ Partial breakpoint values
9. ✅ Multiple variant options with bracket syntax

### Changeset

Created `.changeset/fix-recipe-bracket-syntax.md` documenting the fix for
release notes.

## Build Scripts

### Core Commands

```bash
pnpm build              # Build all packages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chakra-ui/chakra-ui](https://github.com/chakra-ui/chakra-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
