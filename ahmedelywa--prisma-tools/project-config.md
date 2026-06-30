---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PalJS is a comprehensive toolkit for building NodeJS, Prisma, GraphQL, and React applications. It's organized as a monorepo using bun workspaces, providing code generation, admin interfaces, and query optimization tools.

**Current version**: v9 (beta) — Prisma 7 compatible, native Prisma generator.

## Commands

### Development Commands

```bash
# Install dependencies (using bun)
bun install

# Add new packages
bun add [package-name]
bun add -D [dev-package-name]

# Add packages to specific workspace
bun add [package-name] --filter @paljs/[workspace-name]

# Build all v9 packages in dependency order
bun run build

# Run tests (excludes Playwright E2E specs)
bun run test

# Lint and format code with biome
bun run check        # Check for issues
bun run check:fix    # Auto-fix issues
bun run lint         # Lint only
bun run format       # Format code
bun run format:ci    # Check formatting (CI)

# Generate documentation
bun run docs:gen
```

### Package-Specific Build

Individual packages can be built using:
```bash
bun run --filter @paljs/[package-name] build
```

### Testing

- Run all tests: `bun run test`
- Tests use bun's built-in test runner (`bun:test`)
- Admin package uses Vitest
- Test files follow the pattern `*.test.ts`
- Playwright E2E specs (`*.spec.ts`) in `examples/admin-test/e2e/` are excluded from `bun run test` — run them with `npx playwright test` from `examples/admin-test/`
- Snapshots are used extensively for generator output validation
- E2E generator tests create temp projects with symlinked `node_modules` from the monorepo root

### Publishing

- Uses **Changesets** for versioning only, and **bun publish** for publishing
- Currently in **pre-release mode** (`beta` tag) — see `.changeset/pre.json`
- Publishing workflow:
  1. Add changeset: `bunx changeset` (select packages and bump type)
  2. Version bump: `bunx changeset version`
  3. Build packages: `bun run build`
  4. Publish each package in order (plugins → nexus → generator → admin):
     ```bash
     cd packages/plugins && bun publish --otp <otp>
     cd packages/nexus && bun publish --otp <otp>
     cd packages/generator && bun publish --otp <otp>
     cd packages/admin && bun publish --otp <otp>
     ```
- **Why bun publish?** `bunx changeset publish` does NOT resolve `workspace:*` refs with bun. Only `bun publish` properly resolves workspace protocol to actual versions during publish.
- **Important**: The `@paljs/admin` package has `publishConfig.directory: "./dist"` — it publishes from `dist/package.json`. You MUST run `bun run build` after version bump to update the version in `dist/package.json`.
- **Workspace protocol**: During development, `workspace:*` links to local folders (changes reflect immediately). During `bun publish`, it resolves to actual version numbers.
- To exit pre-release mode for stable release: `bunx changeset pre exit`

## Code Architecture

### Monorepo Structure

The project uses bun workspaces with packages in `/packages` directory:

1. **Code Generation** (`packages/generator`)
   - Prisma 7 native generator using `@prisma/generator-helper`
   - Binary: `paljs-generator` (via `bin/cli.js`)
   - Generates Nexus GraphQL types, queries, and mutations
   - Generates client-side `.graphql` files with fragments
   - Generates Admin UI pages and schema
   - Configured via `paljs.config.ts` with `defineConfig()`
   - Writers: DMMF (`writers/dmmf.ts`), types (`writers/types.ts`), Nexus (`writers/nexus/`), GraphQL (`writers/graphql/`), Admin (`writers/admin/`)
   - Config system: `config/define.ts` (defineConfig), `config/loader.ts` (resolution), `config/types.ts` (types)

2. **GraphQL Runtime**
   - `nexus` - Nexus plugin for Prisma integration (PrismaSelect)
   - `plugins` - GraphQL plugins for query optimization (typed PrismaSelect)

3. **UI Components**
   - `admin` - React 19 admin UI components with Tailwind CSS 4, @tanstack/react-table v8, @dnd-kit/sortable
   - Use `bunx shadcn add [component-name]` to add shadcn components

### Key Architectural Patterns

1. **Generator Architecture** (`packages/generator`)
   - Native Prisma generator with `generatorHandler({ onManifest, onGenerate })`
   - Config normalization: `generateGraphQL: true` resolves to `{ nexus: true, nexusOutput: './nexus', client: false, clientOutput: './graphql' }`
   - `generateAdmin: true` resolves to `{ enabled: true, output: 'admin', routerType: 'app' }`
   - Admin schema writer: relation fields (`kind === 'object'`) get `create: false` and `update: false`
   - Per-model configuration for exclusions and customization

2. **Plugin System** (`packages/plugins`)
   - Field selection optimization for GraphQL queries
   - Extensible plugin architecture

3. **Admin UI** (`packages/admin`)
   - React 19 components with TypeScript
   - Tailwind CSS 4 for styling
   - GraphQL integration for CRUD operations
   - Form generation based on Prisma schema

### Prisma 7 Compatibility

- Prisma 7 removes `url` from `datasource` block in `schema.prisma` — connection config goes in `prisma.config.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AhmedElywa/prisma-tools](https://github.com/AhmedElywa/prisma-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
