---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and Cursor when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and Cursor when working with code in this repository.

## Project Structure

Payload is a monorepo structured around Next.js, containing the core CMS platform, database adapters, plugins, and tooling.

### Key Directories

- `packages/` - All publishable packages
  - `packages/payload` - Core Payload package containing the main CMS logic
  - `packages/ui` - Admin UI components (React Server Components)
  - `packages/next` - Next.js integration layer
  - `packages/db-*` - Database adapters (MongoDB, Postgres, SQLite, Vercel Postgres, D1 SQLite)
  - `packages/drizzle` - Drizzle ORM integration
  - `packages/kv-redis` - Redis key-value store adapter
  - `packages/richtext-*` - Rich text editors (Lexical)
  - `packages/storage-*` - Storage adapters (S3, Azure, GCS, Vercel Blob, R2)
  - `packages/email-*` - Email adapters (Nodemailer, Resend)
  - `packages/plugin-*` - Additional functionality plugins
  - `packages/graphql` - GraphQL API layer
  - `packages/translations` - i18n translations
- `test/` - Test suites organized by feature area. Each directory contains a granular Payload config and test files
- `docs/` - Documentation (deployed to payloadcms.com)
- `tools/` - Monorepo tooling
- `templates/` - Production-ready project templates
- `examples/` - Example implementations

### Architecture Notes

- Payload 3.x is built as a Next.js native CMS that installs directly in `/app` folder
- UI is built with React Server Components (RSC)
- Database adapters use Drizzle ORM under the hood
- Packages use TypeScript with strict mode and path mappings defined in `tsconfig.base.json`
- Source files are in `src/`, compiled outputs go to `dist/`
- Monorepo uses pnpm workspaces and Turbo for builds

## Quick Start

1. `pnpm install`
2. `pnpm run build:core`
3. `pnpm run dev` (MongoDB) or `pnpm run dev:postgres`

## Build Commands

- `pnpm install` - Install all dependencies
- `pnpm turbo` - All Turbo commands should be run from root with pnpm - not with `turbo` directly
- `pnpm run build` or `pnpm run build:core` - Build core packages (excludes plugins and storage adapters)
- `pnpm run build:all` - Build all packages
- `pnpm run build:<directory_name>` - Build specific package (e.g. `pnpm run build:db-mongodb`, `pnpm run build:ui`)

## Development

### Coding Patterns and Best Practices

- Always use object parameters for function arguments: `fn({ name }: { name: string })` not `fn(name: string)` (improves backwards-compatibility)
- Prefer types over interfaces (except when extending external types)
- Prefer functions over classes (classes only for errors/adapters)
- Prefer pure functions; when mutation is unavoidable, return the mutated object instead of void.
- Organize functions top-down: exports before helpers
- Use JSDoc for complex functions; add tags only when justified beyond type signature
- Use `import type` for types, regular `import` for values, separate statements even from same module
- Prefix booleans with `is`/`has`/`can`/`should` (e.g., `isValid`, `hasData`) for clarity
- Prefer self describing function and variable names over generic names with comments to explain their purpose
- **Translation/Label handling**: Always use `getTranslation` from `@payloadcms/translations` when you need to render labels defined in the config - it already handles functions, strings, and translation objects correctly. Don't write custom if/else logic to handle different label types.
- **Memoize arrays/objects passed to hooks**: Never pass inline array/object literals to custom hooks - they create new references on every render, breaking memoization and causing unnecessary re-renders or remounts.

  ```typescript
  // BAD - creates new array every render, breaks hook memoization
  const [Drawer] = useHierarchyDrawer({
    filterByCollection: [collectionSlug],
  })

  // GOOD - memoized, stable reference
  const filterByCollection = useMemo(() => [collectionSlug], [collectionSlug])
  const [Drawer] = useHierarchyDrawer({
    filterByCollection,
  })
  ```

- Commenting Guidelines

  - Execution flow: Skip comments when code is self-documenting. Keep for complex logic, non-obvious "why", multi-line context, or if following a documented, multi-step flow.
  - Top of file/module: Use sparingly; only for non-obvious purpose/context or an overview of complex logic.
  - Type definitions: Property/interface documentation is always acceptable.

- Logger Usage (`payload.logger.error`)
  - Valid: `payload.logger.error('message')` or `payload.logger.error({ msg: '...', err: error })`
  - Invalid: `payload.logger.error('message', err)` - don't pass error as second argument
  - Use `err` not `error`, use `msg` not `message` in object form

### React Component File Structure

Each React component should have its own named folder:

```
ComponentName/
├── index.tsx       # Component implementation
└── index.scss      # Styles (if applicable)
```

- **Do:** Create a folder per component with `index.tsx` and `index.scss`
- **Don't:** Place multiple `ComponentName.tsx` files in a single folder with one shared `.scss` file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [payloadcms/payload](https://github.com/payloadcms/payload) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
