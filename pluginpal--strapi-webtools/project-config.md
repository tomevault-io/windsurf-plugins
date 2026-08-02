---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Strapi Webtools is a plugin for Strapi CMS v5 that provides URL management, routing, and an extensible addon system. It's a monorepo managed with Yarn workspaces and Turborepo, containing:

- **packages/core**: Main plugin (`strapi-plugin-webtools`)
- **packages/cli**: Installation CLI tool (`webtools-cli`)
- **packages/addons/sitemap**: Sitemap addon example
- **packages/docs**: Documentation site
- **playground**: Development Strapi instance for testing

## Development Setup

### Initial Setup

```bash
# Install root dependencies
yarn install

# Install playground dependencies (runs automatically via postinstall)
yarn playground:install
```

### Development Workflow

Development requires running **two terminal sessions**:

**Terminal 1 - Build the plugin in watch mode:**
```bash
yarn develop
```
This runs TypeScript compilation in watch mode across all packages and uses `yalc` to link them.

**Terminal 2 - Run the playground Strapi instance:**
```bash
yarn playground:develop
```
This starts Strapi at http://localhost:1337 with the plugin pre-installed.

Changes to the plugin code will rebuild automatically (Terminal 1) and hot-reload in the playground (Terminal 2).

### Building for Production

```bash
yarn build
```
Builds all packages using Turborepo's caching and dependency graph.

## Testing

### Node version

Always activate Node 20 before running yarn commands:

```bash
nvm use  # reads .nvmrc (Node 20)
```

The repo uses **Yarn 4.11.0** via `yarnPath` in `.yarnrc.yml`. Running `yarn` anywhere inside the repo automatically uses this version — even in subdirectories like `playground/`.

### Unit Tests

```bash
# Run all unit tests
yarn test:unit

# Run unit tests in watch mode (add to jest command in package.json)
ENV_PATH=./playground/.env jest --watch
```

Unit tests are located in `__tests__` directories:
- `packages/core/server/middlewares/__tests__/`
- `packages/core/server/hooks/__tests__/`
- `packages/core/server/controllers/__tests__/`
- `packages/addons/sitemap/server/utils/__tests__/`

Uses Jest with ts-jest preset. Test files: `*.test.ts` or `*.test.js`

`playground/.env` is **force-committed** to git with placeholder values (`tobemodified`) so CI can boot Strapi. Do not add real secrets to it — use your local `.env` override for `WEBTOOLS_LICENSE_KEY` and similar.

### Integration Tests

```bash
yarn test:integration
```
Runs healthcheck integration test against the playground instance.

### E2E Tests

```bash
yarn test:e2e
```
Opens Cypress test runner. E2E tests have `.cy.ts` or `.cy.tsx` extensions.

## Code Quality

### Linting

```bash
# Check all packages
yarn eslint

# Auto-fix issues
yarn eslint:fix

# Type checking (no emit)
yarn tscheck
```

Uses `@uncinc/eslint-config` with special overrides for Cypress and Jest files.

## Architecture Overview

### Core Plugin Structure

The plugin follows Strapi 5's plugin architecture with two entry points:

**Server-side** (`packages/core/server/index.ts`):
- `register()`: Registers services, controllers, routes, content types
- `bootstrap()`: Registers document middlewares and hooks
- `contentTypes`: Defines `url-alias` and `url-pattern` internal content types
- `routes`: Admin API and public Content API endpoints
- `services`: Business logic (url-alias, url-pattern, info)
- `middlewares`: Document lifecycle hooks for automatic URL generation
- `controllers`: Request handlers for API endpoints

**Admin-side** (`packages/core/admin/index.ts`):
- `register()`: Registers the plugin with Strapi Admin
- `bootstrap()`: Injects UI into Content Manager and Content-Type Builder
- `permissions`: RBAC permission definitions

### Key Architectural Concepts

#### 1. URL Aliases and Patterns

**URL Pattern** (`plugin::webtools.url-pattern`):
- Template that defines how URLs are generated for a content type
- Uses bracket syntax: `[fieldName]`, `[relation.field]`, `[pluralName]`, `[documentId]`
- Example: `/blog/[category.slug]/[title]` → `/blog/news/hello-world`
- Stored in database, managed via Admin UI

**URL Alias** (`plugin::webtools.url-alias`):
- The actual generated URL path for an entry
- Localized (supports i18n)
- Tracks whether it was generated or manually set (`generated` field)
- Automatically created/updated by middlewares

#### 2. Document Middleware Chain

Three middlewares hook into the Strapi 5 document lifecycle:

1. **generate-url-alias.ts**: Runs on create/update/clone
   - Fetches URL patterns for the content type
   - Resolves pattern templates using entry data
   - Creates/updates URL alias records
   - Respects manual URLs (`generated: false`)

2. **prevent-duplicate-urls.ts**: Ensures URL uniqueness
   - Appends numeric suffixes when conflicts exist (-1, -2, etc.)

3. **delete-url-alias.ts**: Cleans up on entry deletion

These run BEFORE/AFTER document operations using Strapi's document middleware API (not legacy entity service).

#### 3. Content Type Enablement

Content types opt into Webtools via `pluginOptions`:

```javascript
{
  "pluginOptions": {
    "webtools": { "enabled": true }
  }
}
```

When enabled:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pluginpal/strapi-webtools](https://github.com/pluginpal/strapi-webtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
