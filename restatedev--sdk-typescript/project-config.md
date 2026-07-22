---
trigger: always_on
description: This document provides guidance for AI coding assistants (like Claude, Cursor, etc.) working with this TypeScript monorepo.
---

# AI Agent Guide

This document provides guidance for AI coding assistants (like Claude, Cursor, etc.) working with this TypeScript monorepo.

## ⚠️ Critical: Interactive Commands

**DO NOT RUN these commands directly - they will hang:**

- `pnpm new` - Package generator
- `pnpm delete` - Package deletion
- `pnpm add-entry` - Add custom entry points
- `pnpm changeset` - Create changesets

**Instead:** Ask the user to run these commands and wait for them to complete. Provide clear instructions on what options to select.

**Safe to run (non-interactive):**
- `pnpm install`, `pnpm build`, `pnpm test`, `pnpm verify`
- `pnpm dev`, `pnpm lint`, `pnpm format`
- `pnpm version` (applies changesets)
- `pnpm generate:configs` (regenerates TypeScript configs)

## Project Overview

This is a TypeScript monorepo template built with:
- **PNPM Workspaces** for package management
- **Turbo** for task orchestration and caching
- **tsdown** for building publishable packages
- **Vitest** for testing
- **Changesets** for version management

## Key Concepts

### Package Types

1. **Public Libraries** (`packages/libs/*/` with `private: false`)
   - Built with tsdown to `dist/`
   - Published to npm
   - Support ESM + CJS + TypeScript declarations
   - Can have multiple entry points (subpath exports)

2. **Private Libraries** (`packages/libs/*/` with `private: true`)
   - Source-only (no build step)
   - Used internally, bundled into public packages
   - Main points to `./src/index.ts`

3. **Test Packages** (`packages/tests/*/`)
   - Always private
   - Use Vitest for testing
   - Can test libraries from source (dev) or built output (CI)

4. **Example Packages** (`packages/examples/*/`)
   - Always private
   - Demonstrate library usage
   - Can run from source (dev) or built output (production)

### Build System

- **Dev Mode**: No builds required! Type checking only with `tsc --noEmit --watch`
- **Production Mode**: Full builds with tsdown (ESM + CJS + declarations)
- **Turbo**: Automatically handles dependency ordering and caching
- **Path Mappings**: Auto-generated in root `tsconfig.json` for IDE support

### Important Files

- `.templates/` - Plop templates for generating packages
- `turbo.json` - Task configuration with dependency graph
- `pnpm-workspace.yaml` - Workspace configuration with catalogs
- `tsconfig.base.json` - Shared TypeScript config
- `vitest.config.ts` - Shared test configuration
- `api-extractor.base.json` - API validation configuration

## Common Tasks

### Creating a New Package

**⚠️ IMPORTANT: This is an interactive command - DO NOT RUN IT DIRECTLY**

The `pnpm new` command is interactive and will hang if run by an AI agent. Instead:

1. **Ask the user to run it:**
   ```
   "Please run `pnpm new` to create a new package. I'll help you configure it after."
   ```

2. **Or inform the user what needs to be created and let them decide:**
   ```
   "To add a new library package, you'll need to run `pnpm new` and select:
   - Package type: lib
   - Package name: my-package
   - Private: no (for publishable) or yes (for internal)
   
   Would you like me to wait while you run this, or would you prefer to do it later?"
   ```

**Never attempt to run interactive commands like:**
- `pnpm new` - Package generator (interactive prompts)
- `pnpm delete` - Package deletion (interactive selection)
- `pnpm add-entry` - Entry point addition (interactive prompts)
- `pnpm changeset` - Changeset creation (interactive prompts)

### Adding Package Dependencies

**For monorepo dependencies:**
```bash
cd packages/libs/my-package
pnpm add "@restatedev/other-package@workspace:*"
```

**For publishable packages depending on other publishable packages:**
Add to `tsdown.config.ts`:
```typescript
export default defineConfig({
  // ...
  external: ["@restatedev/other-package"],
});
```

This prevents bundling the dependency into your package.

### Modifying Package Scripts

All package scripts use Turbo with `--filter={.}...` pattern:
```json
{
  "scripts": {
    "build": "turbo run _build --filter={.}...",
    "_build": "tsc --noEmit && tsdown"
  }
}
```

- Public-facing scripts (e.g., `build`) use Turbo with filters
- Internal scripts (e.g., `_build`) are the actual commands
- The `--filter={.}...` pattern means "this package and its dependencies"

### Running Checks

**Before committing:**
```bash
pnpm verify
```

This runs:
- Format check (`prettier`)
- Lint (`eslint`)
- Type check (`tsc --noEmit`)
- Build all packages
- Tests
- Export validation (ATTW)
- API validation (API Extractor)

### Adding Custom Entry Points

**⚠️ INTERACTIVE COMMAND - Ask user to run it**

The `pnpm add-entry` command is interactive. Tell the user:
```
"To add a custom entry point (e.g., @restatedev/pkg/utils), please run:
  pnpm add-entry

This will automatically update:
- package.json exports and typesVersions
- tsdown.config.ts entry array
- Root tsconfig.json paths
- API Extractor configs
- Package scripts"
```

## Configuration Patterns

### TypeScript Config Layers

**Root level:**
- `tsconfig.base.json` - Shared compiler options
- `tsconfig.json` - Extends base + path mappings (auto-generated)

**Package level:**
- `tsconfig.json` - Extends root (inherits path mappings)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [restatedev/sdk-typescript](https://github.com/restatedev/sdk-typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
