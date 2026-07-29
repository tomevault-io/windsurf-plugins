---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# BootstrapVueNext Development Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Required Setup

- Install Node.js 20.x or 22.x (engine requires >=22.0.0 but 20.x works with warnings)
- Install pnpm globally: `npm install -g pnpm@10.13.1` (ONLY pnpm is allowed as package manager) Use the version specified in the packageManager field of the package.json file
- Clone repository and open the **root** directory (not subdirectories like packages/bootstrap-vue-next)

### Bootstrap, Build, and Test the Repository

1. `pnpm install --ignore-scripts --frozen-lockfile` -- takes 2 seconds. Use `--ignore-scripts` to avoid docs build network issues.
2. `pnpm --filter bootstrap-vue-next run build` -- takes 27 seconds. NEVER CANCEL. Set timeout to 60+ minutes.
3. `pnpm --filter bootstrap-vue-next run test:unit:ci` -- takes 40 seconds. NEVER CANCEL. Set timeout to 60+ minutes.
4. `pnpm --filter bootstrap-vue-next run test:lint` -- takes 12 seconds. NEVER CANCEL. Set timeout to 30+ minutes.

### Build Individual Components

- Bootstrap Vue Next core package: `pnpm --filter bootstrap-vue-next run build` -- 27 seconds
- Nuxt package: `pnpm --filter @bootstrap-vue-next/nuxt run build` -- 25 seconds
- Playground app: `pnpm --filter playground run build` -- 8 seconds
- **NEVER** try to build docs app directly - it fails due to network connectivity (OpenCollective API)

### Development Servers

- Core package dev server: `pnpm --filter bootstrap-vue-next run dev` (runs on <http://localhost:5174>)
- Playground app dev server: `pnpm --filter playground run dev` (runs on <http://localhost:5173>)
- Docs dev server: `pnpm --filter docs run dev` (runs on <http://localhost:8000>)
- All dev servers: `pnpm dev` (starts all development environments in parallel)

### Exporting public files in the main bootstrap-vue-next package

The library uses a custom export structure that must fit a specific pattern to ensure proper tree-shaking and module resolution. For the build process, as shown in the vite.config.ts file, it resolves the directories and files. The purpose of this is to allow for both high-level path imports, as well as more specific (and better tree-shaken) imports of individual components, composables, directives, and types. Ex: `import {BButton} from 'bootstrap-vue-next/components/button'` and `bootstrap-vue-next/components` are both valid, instead of `import {BButton} from 'bootstrap-vue-next'` which would import the entire package.

The rules of how this functions is that files that are publicly exported must be in their own directory, with an index.ts file that exports the relevant items. The resolution of this build process is in the vite.config.ts file. Then the package.json file uses the "exports" field to map the paths to the built files.

If the file is intended to be public, it must follow this pattern. For example, if you are adding a new composable, it must be in its own directory under `src/composables/` with an index.ts file that exports the composable. Then you must add the relevant entry to the "exports" field in the package.json file. It must also be exported from the main `src/composables/index.ts` file. This ensures that the composable can be imported both from the high-level path and the specific path for best tree-shaking.

Private files should exist in the root of the domain they are related to. For example, utility functions for composables should be in the `src/composables/` directory but not exported in an index.ts file. This keeps the public API clean and ensures that only intended files are accessible to users of the library.

**Consuming imports for tree-shaking**: When importing components, composables, or directives from bootstrap-vue-next in demo files, documentation, or user code, always use the specific paths for optimal tree-shaking:

**Exception** for types, which can be imported from the root path without tree-shaking issues since they are removed at compile time. For example, `import type {BButtonProps} from 'bootstrap-vue-next'` is acceptable

- Components:
  <<< FRAGMENT ./demo/ImportExamples.ts#components{typescript}
- Type-only imports:
  <<< FRAGMENT ./demo/ImportExamples.ts#types{typescript}
- Composables:
  <<< FRAGMENT ./demo/ImportExamples.ts#composables{typescript}
- Directives:
  <<< FRAGMENT ./demo/ImportExamples.ts#directives{typescript}

Avoid importing from the root package (`'bootstrap-vue-next'`) as this imports the entire library and defeats tree-shaking.

The fault of not properly following this structure will lead to build errors or improper module resolution.

## Validation

### Building documentation and checking doc files when adding new props

When you add new props to the core packages' components, you must add those props in the .data.ts files from the documentation for that component. When you 
add new props to the core packages' components, you must build the documentation to ensure no type-checking errors have occurred.

### Always Validate Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bootstrap-vue-next/bootstrap-vue-next](https://github.com/bootstrap-vue-next/bootstrap-vue-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
