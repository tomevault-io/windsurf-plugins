---
trigger: always_on
description: Quagga2 is a barcode scanner library written in TypeScript that bundles all dependencies with Webpack. The project produces standalone browser (`dist/quagga.min.js`) and Node.js (`lib/quagga.js`) builds.
---

# GitHub Copilot Instructions for Quagga2

## Repository Overview

Quagga2 is a barcode scanner library written in TypeScript that bundles all dependencies with Webpack. The project produces standalone browser (`dist/quagga.min.js`) and Node.js (`lib/quagga.js`) builds.

## Key Architecture Principles

1. **All dependencies are bundled** - Consumers never install our dependencies directly, so everything goes in `devDependencies`
2. **TypeScript 5.9.3** - We use modern TypeScript with strict type checking
3. **Webpack 4** - Pinned due to compatibility constraints (v5 migration not yet done)
4. **Old test tooling** - mocha@5, chai@4 are pinned (newer versions have breaking changes)

## Critical Files to Maintain

### DEPENDENCIES.md

**Always check if `DEPENDENCIES.md` needs updating when:**

- Adding, removing, or upgrading any npm package
- Changing what packages are imported in `src/` files
- Modifying build configuration (webpack, babel)
- Discovering security vulnerabilities or applying overrides
- Changing version pinning strategy in `.ncurc.json`

**Update these sections:**
- "Runtime Code Dependencies" - if adding/removing imports
- "Build & Development Tools" - if adding dev tooling
- "Overrides" - if modifying `package.json` overrides field
- "Bundle Size Impact" - if adding runtime dependencies
- "Security Considerations" - if addressing CVEs
- "Version Constraints" - if pinning versions in `.ncurc.json`
- "Last updated" date at bottom

### .ncurc.json

This file controls which packages `npm-check-updates` can upgrade:

```json
{
  "reject": ["mocha", "@types/mocha", "webpack", "webpack-cli", "cypress", "@cypress/*", "babel-loader", "source-map-loader", "chai", "eslint"]
}
```

**When adding to reject list:**
- Document the reason in `DEPENDENCIES.md` → "Version Constraints" → "Pinned Versions"
- Explain what breaks if the package is upgraded

### package.json

**Special considerations:**
- All dependencies go in `devDependencies` (we bundle everything)
- `overrides` field is used for security fixes of transitive dependencies
- Version pinning is handled by `.ncurc.json`, not strict versions here

## Development Workflow

### Before Making Changes

1. Run `git status` to check current branch (should be on `dev` or feature branch, not `master`)
2. Run `npm run check-types` to verify TypeScript compiles
3. Run `npm run lint` to check code style

### When Adding Dependencies

1. **Determine dependency type:**
   - Is it imported in `src/` code? → Runtime dependency (but still goes in `devDependencies`)
   - Is it only used by build tools? → Build tool (in `devDependencies`)

2. **Check bundle impact:**
   ```bash
   npm install --save-dev <package>
   npm run build:prod
   # Check size of dist/quagga.min.js before and after
   ```

3. **Update documentation:**
   - Add entry to `DEPENDENCIES.md` under appropriate section
   - If pinning version, add to `.ncurc.json` reject list and document why

4. **Test thoroughly:**
   ```bash
   npm run build-and-test
   ```

### When Upgrading Dependencies

1. **Use upgrade script:**
   ```bash
   npm run upgrade-deps   # Automated upgrade with testing (use cautiously)
   ```

2. **Check rejected packages:**
   - Review `.ncurc.json` reject list
   - Verify if any can be safely upgraded now
   - Update `DEPENDENCIES.md` if upgrading previously-pinned packages

3. **Test extensively:**
   - Run full test suite: `npm run build-and-test`
   - Test example files in browser: `docs/examples/live_w_locator.html`
   - Check Cypress tests: `npm run cypress:run`

### When Removing Dependencies

1. **Verify it's unused:**
   ```bash
   grep -r "from 'package-name'" src/
   grep -r 'from "package-name"' src/
   grep -r "require('package-name')" src/
   ```

2. **Check transitive usage:**
   ```bash
   npm ls package-name  # See if other packages depend on it
   ```

3. **Update documentation:**
   - Remove from `DEPENDENCIES.md`
   - Remove from `.ncurc.json` if it was pinned
   - Update bundle size table if it was a runtime dependency

## Code Style Guidelines

### TypeScript

- **Avoid `as any`** - Use specific type assertions like `as Uint8ClampedArray<ArrayBuffer>`
- **Use proper generics** - ArrayBuffer generics required for TS 5.x compatibility
- **Document complex types** - Add JSDoc comments for non-obvious type casts

### Imports

- **Prefer ES modules** - Use `import` not `require()` in new code
- **Tree-shakeable imports** - Use `import merge from 'lodash/merge'` not `import { merge } from 'lodash'`
- **Type-only imports** - Use `import type` when importing only types

### File Organization

- **Tests colocated** - Test files live in `src/**/test/` next to code being tested
- **Type definitions** - Global types in `src/global.d.ts`, package types in `src/vendor.d.ts`
- **Config files** - Build configs in `configs/`, not root directory

## Testing Requirements

### CRITICAL: NODE_OPTIONS Flag

**⚠️ IMPORTANT:** Cypress tests MUST be run with `NODE_OPTIONS=--openssl-legacy-provider` due to Node.js 17+ security changes disabling legacy OpenSSL algorithms. Our build dependencies require these algorithms.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericblade/quagga2](https://github.com/ericblade/quagga2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
