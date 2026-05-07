---
trigger: always_on
description: This is a **Yarn 4 monorepo** publishing `@gfazioli/mantine-list-view-table`, a Finder-style table component for Mantine 7+. Two workspaces:
---

# Mantine ListViewTable - AI Coding Agent Instructions

## Project Overview

This is a **Yarn 4 monorepo** publishing `@gfazioli/mantine-list-view-table`, a Finder-style table component for Mantine 7+. Two workspaces:
- `package/` - React component library with dual ESM/CJS build
- `docs/` - Next.js 15 static documentation site using `@mantinex/demo`

**Key external dependencies:**
- Mantine >=7.0.0 (core, hooks) - peer dependency (dev: 8.3.12)
- React 18.x or 19.x - peer dependency (dev: 19.2.3)
- @tabler/icons-react ^3.34.0 - peer dependency for icons

## Architecture

### Build System

**Library Build (Rollup):**
```bash
yarn build  # Runs: rollup → generate-dts → prepare-css
```
- **Input:** `package/src/index.ts`
- **Output:** Dual format in `package/dist/`
  - `esm/` - ES modules (`.mjs`)
  - `cjs/` - CommonJS (`.cjs`)
  - `types/` - TypeScript declarations (`.d.ts`, `.d.mts`)
  - `styles.css` and `styles.layer.css` - PostCSS processed
- **CSS Modules:** Scoped with hash prefix `me` (via `hash-css-selector`)
- **'use client' directive:** Auto-injected for Next.js App Router compatibility

**Docs Build (Next.js):**
```bash
yarn dev          # Development server with HMR
yarn docs:build   # docgen → next build (static export)
yarn docs:deploy  # Build → gh-pages deploy
```
- **Base path:** Auto-configured from `package/package.json` repository field
- **MDX:** Enabled with `remark-slug` for heading anchors
- **Demo system:** Uses `@mantinex/demo` for interactive examples

### Component Structure

**Single-file pattern:** Each feature lives in one file (e.g., `ListViewTable.tsx` is 1199 lines). No fragmentation unless truly necessary.

**Key component concepts:**
1. **Column management:** Draggable reordering, resizable widths, configurable min/max constraints
2. **State control:** Internal or external - component accepts both controlled/uncontrolled patterns
3. **Sticky behavior:** Supports sticky headers and sticky identifier columns in scroll containers
4. **Custom rendering:** `renderCell`, `renderHeader`, `cellStyle` props for per-column customization
5. **Empty/loading states:** Rich configuration - from simple text to full component rendering

## Development Workflows

### Testing

**Run tests:**
```bash
yarn jest        # Run Jest tests only
yarn test        # Full suite: syncpack + prettier + typecheck + lint + jest
```

**Test setup:**
- **Runner:** Jest 29 with jsdom environment
- **Transform:** `esbuild-jest` (fast, no babel)
- **Utilities:** `@mantine-tests/core` provides custom render utilities
- **CSS mocking:** `identity-obj-proxy` for CSS modules
- **Setup:** `jsdom.mocks.cjs` for browser API mocks

**Test conventions:**
- Use `@mantine-tests/core` render instead of RTL directly
- Mock data interfaces at top of test file
- Test empty states, loading states, and custom renderers
- Example: `ListViewTable.test.tsx` shows standard patterns

### Type Checking & Linting

```bash
yarn typecheck    # TSC (no emit) + docs typecheck
yarn lint         # eslint + stylelint
yarn prettier:check  # Or prettier:write
```

**TypeScript configs:**
- `tsconfig.json` - Main config (includes package/, scripts/, @types/)
- `tsconfig.build.json` - Build-specific config for Rollup
- `tsconfig.eslint.json` - ESLint parser config

**Linting:**
- ESLint with `eslint-config-mantine`, React, a11y plugins
- Stylelint for CSS with caching
- All configs use flat config format (ESM)

### Demo Creation

**Critical pattern:** Demos MUST follow the `@mantinex/demo` structure:

```tsx
import { ListViewTable } from '@gfazioli/mantine-list-view-table';
import { MantineDemo } from '@mantinex/demo';

function Demo() {
  return <ListViewTable columns={columns} data={data} rowKey="id" />;
}

const code = `
import { ListViewTable } from '@gfazioli/mantine-list-view-table';
// User-facing import path ^^^

function Demo() {
  return <ListViewTable columns={columns} data={data} rowKey="id" />;
}
`;

// Named export (NOT default)
export const demoName: MantineDemo = {
  type: 'code', // or 'configurator'
  component: Demo,
  code: [{ fileName: 'Demo.tsx', code, language: 'tsx' }],
};
```

**Dual export pattern for data/columns:**
```tsx
// Export BOTH code string and actual values
export const dataCode = `export const data = [...]`;
export const data = [...];
```

**File naming:** `ComponentName.demo.feature.tsx` (e.g., `ListViewTable.demo.ellipsis.tsx`)

**Configurator demos:**
- Component receives props via function parameter: `Demo(props: Omit<Props, 'children'>)`
- Code template uses `{{props}}` placeholder
- Controls array defines UI with types: `boolean`, `select`, `segmented`, `color`, `number`, `string`

See: `.github/skills/react-ts-library-monorepo/assets/templates/` for complete examples

### Release Process

```bash
yarn release:patch  # Bump patch version, build, publish, deploy docs
yarn release:minor  # Bump minor version
yarn release:major  # Bump major version
```

**Release workflow:**
1. Runs `scripts/release.ts` with semantic versioning
2. Updates `package/package.json` version
3. Git commit + tag
4. npm publish (requires auth)
5. Builds and deploys docs to gh-pages

**Documentation generation:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gfazioli/mantine-list-view-table](https://github.com/gfazioli/mantine-list-view-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
