---
trigger: always_on
description: Development guide for the explorer component library within the walkerOS
---

# AGENT.md — @walkeros/explorer

Development guide for the explorer component library within the walkerOS
monorepo.

## Project Overview

**@walkeros/explorer** is a React component library for walkerOS documentation
and exploration. It provides interactive demos and editors with Monaco Editor
integration for live code editing, event visualization, and mapping
configuration.

## Code Standards

### Import Statements

- **ALWAYS import modules at the top of files** - Never use inline
  `typeof import()` or dynamic `await import()`
- **Exception — `monaco-editor`**: Never import `monaco-editor` at runtime
  (top-level or `require()`). Monaco accesses `window` at module evaluation
  time, which crashes SSR. Instead:
  - Use `useMonaco()` hook from `@monaco-editor/react` in components (returns
    `null` during SSR, the CDN-loaded instance after load)
  - For utility modules, accept the monaco instance as a parameter (see
    `initMonacoJson()` in `monaco-json-schema.ts`)
  - `import type` from `monaco-editor` is always safe (erased at compile time)
  - `@monaco-editor/react` imports are always safe (SSR-aware internally)
- Use proper type imports: `import type { Monaco } from '@monaco-editor/react';`

## Development Commands

### Essential Commands

```bash
npm test           # Run Jest tests
npm run dev        # Run tests in watch mode
npm run build      # Build package (tsup for JS/TS, SCSS compilation for styles)
npm run lint       # Type check with tsc and lint with ESLint
npm run storybook  # Start Storybook (port 6007)
npm run clean      # Clean build artifacts and dependencies
```

### Testing

- **Run all tests**: `npm test`
- **Watch mode**: `npm run dev`
- **Test files**: Located in `src/__tests__/` and `src/**/__tests__/`
- Test setup uses `@testing-library/react` with custom mocks for Monaco Editor

**CRITICAL: Test Integrity Rules**

- **NEVER create fake/mock tests that pretend to validate real functionality**
- **NEVER use simple string checks to simulate complex behavior** (e.g.,
  checking if a string contains "WalkerOS" to simulate TypeScript type checking)
- **NEVER gaslight the user by making tests pass through deception**
- **If real integration testing is difficult or impossible, EXPLICITLY state
  this limitation to the user**
- **Be honest about what tests actually verify vs what they appear to verify**
- **When faced with complex integration testing (Monaco, browser APIs, etc.),
  ask the user how to proceed rather than faking it**
- **If something isn't working and you don't know why, SAY SO IMMEDIATELY - do
  not keep trying random solutions**
- **Do not claim success based on passing tests unless those tests actually
  validate the user's requirements**
- **String manipulation tests (contains, regex) are NOT integration tests - they
  only verify string content**

Tests must reflect reality. A passing test should mean the feature actually
works, not that we fooled ourselves.

## When You Don't Know

**If you're stuck or something doesn't work:**

1. **Admit it immediately** - "I don't know why this isn't working"
2. **Show what you've tried** - Be transparent about the attempts
3. **Ask for help** - "Can you check X in the browser console?" or "Should we
   try a different approach?"
4. **Do NOT keep iterating on solutions** without user feedback
5. **Do NOT claim progress** when the actual requirement still fails

It is ALWAYS better to say "I don't know" than to waste the user's time with
false solutions.

### Building

Build creates:

- `dist/index.js` (CJS) and `dist/index.mjs` (ESM) - main module
- `dist/index.d.ts` - TypeScript declarations
- `dist/styles.css` - compiled SCSS styles

The build is configured in `tsup.config.ts` with SCSS compilation via Sass.

### Bundling Dependencies

**Important**: When adding new dependencies that should be bundled into explorer
(not resolved from consumer's node_modules), add them to the `noExternal` array
in `tsup.config.ts`.

Currently bundled:

- `clsx` - Class name utility
- `tailwind-merge` - Tailwind class merging
- `@iconify/react` - Icon component library

**Why bundle these?** Explorer is used via symlink in the walkerOS monorepo. If
a dependency is externalized (the default), it must exist in the consumer's
node_modules. Bundling ensures explorer works without requiring consumers to
install these dependencies.

**When to bundle vs externalize:**

- **Bundle** (`noExternal`): Small utilities, UI libraries that are
  implementation details
- **Externalize** (`external`): Large dependencies consumers likely have (React,
  Monaco), peer dependencies, walkerOS packages

## Architecture

### Component Hierarchy (Atomic Design)

The codebase strictly follows **Atomic Design** principles:

1. **Atoms** (`src/components/atoms/`): Base UI elements
   - `Box`, `Button`, `ButtonGroup`, `Header`, `Toggle`
   - Mapping primitives: `mapping-string`, `mapping-number`, `mapping-boolean`
   - Form controls: `icon-button`, `mapping-collapsible`, `field-header`

2. **Molecules** (`src/components/molecules/`): Component combinations
   - Navigation: `mapping-tab-bar`, `mapping-tree-sidebar`, `mapping-breadcrumb`
   - Pane views: `mapping-*-pane-view` (rule, entity, consent, condition, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elbwalker/walkerOS](https://github.com/elbwalker/walkerOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
