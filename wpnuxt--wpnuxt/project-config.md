---
trigger: always_on
description: Provides base queries for:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

WPNuxt is a Nuxt module that integrates WordPress with Nuxt 4 via GraphQL (WPGraphQL). It generates type-safe composables from GraphQL queries and provides utilities for fetching WordPress content.

## Development Commands

### Setup
```bash
pnpm install                    # Install dependencies
pnpm run dev:prepare           # Build module stub, prepare playgrounds, generate types
```

### Development
```bash
pnpm run dev                   # Run full playground (with @nuxt/ui)
pnpm run dev:core              # Run core playground (minimal setup)
pnpm run dev:blocks            # Run blocks playground
pnpm run dev:build             # Build main playground
pnpm run dev:core:build        # Build core playground
```

### Testing & Linting
```bash
pnpm run test                  # Run Vitest tests
pnpm run test:watch            # Run tests in watch mode
pnpm run test:types            # Type check module and playground
pnpm run lint                  # Run ESLint
```

### Build & Release
```bash
pnpm run prepack               # Build the module for distribution
pnpm run release               # Lint, test, build, create changelog, publish, and push tags
```

## Architecture

### Module Structure

**Core Module (`src/module.ts`)**
- Entry point that configures the WPNuxt module
- Registers the `nuxt-graphql-middleware` dependency and adds a built-in HTML sanitization plugin (`runtime/plugins/sanitizeHtml.ts`) that lazy-loads `dompurify` on the client for the `v-sanitize-html` directive
- Validates configuration (WordPress URL required, no trailing slash)
- Merges default queries from `src/runtime/queries/` with user queries from `extend/queries/`
- Triggers composable generation via `src/generate.ts`

**Composable Generation (`src/generate.ts`)**
- Scans merged `.gql`/`.graphql` files and parses them using `src/utils/useParser.ts`
- For each GraphQL query, generates an auto-imported composable: `use{QueryName}()`
- Regular queries use `useWPContent()`, connection queries (with `pageInfo`) use `useWPConnection()`
- Composable params accept `MaybeRefOrGetter<T>` — plain objects, refs, computed, or getter functions
- Generates TypeScript declarations with proper return types based on fragments
- Output written to `.nuxt/wpnuxt/index.mjs` and `.nuxt/wpnuxt/index.d.ts`

**Runtime Composables**
- `useWPContent()` (`src/runtime/composables/useWPContent.ts`) — wraps `useAsyncGraphqlQuery()` from nuxt-graphql-middleware, extracts nested data using the `nodes` path, supports reactive params with auto-watch, retry, timeout, and SSG caching
- `useWPConnection()` (`src/runtime/composables/useWPConnection.ts`) — wraps `useWPContent()` for connection queries, splits result into `data` (nodes array) and `pageInfo`, provides `loadMore()` for infinite scroll accumulation

**Runtime Components (`src/runtime/components/WPContent.vue`)**
- `<WPContent>` - renders WordPress content with automatic internal link interception
- Uses `BlockRenderer` when `@wpnuxt/blocks` is installed, falls back to `v-sanitize-html`
- Intercepts clicks on internal `<a>` tags and uses `navigateTo()` for client-side navigation
- Props: `node` (content object), `replaceLinks` (per-instance override of global config)

**Auto-imported Utilities**
- `isInternalLink()`, `toRelativePath()` — link detection and conversion (`src/runtime/util/links.ts`)
- `getRelativeImagePath()` — WordPress image URL to relative path (`src/runtime/util/images.ts`)
- `isPage()`, `isPost()`, `isContentType()` — type guards for narrowing `NodeByUri` union types (`src/runtime/util/content-type.ts`)
- `unwrapScalar()`, `unwrapConnection()` — ACF field normalization helpers (`src/runtime/util/acf.ts`)
- `usePrevNextPost()` — previous/next post navigation (`src/runtime/composables/usePrevNextPost.ts`)

### Query System

**Default Queries (`src/runtime/queries/`)**
Provides base queries for:
- Posts (by URI, ID, category)
- Pages (by URI, ID, all pages)
- Menus, Nodes, Revisions, Viewer, GeneralSettings
- Fragments for common WordPress types (Post, Page, ContentNode, etc.)

**Query Merging (`src/utils/index.ts:mergeQueries()`)**
1. Copies default queries from `src/runtime/queries/` to `.queries/` folder
2. If `extend/queries/` exists in the project, copies/overwrites queries from there
3. This merged folder is then scanned to generate composables

**Connection Pattern Detection**
The parser (`src/utils/useParser.ts`) detects the WPGraphQL connection pattern when `pageInfo` and `nodes` appear as sibling fields. When detected, `hasPageInfo` is set on the query, and the generator produces a `useWPConnection()` call instead of `useWPContent()`.

**Extending Queries**
Create `.gql` files in your project's `extend/queries/` folder (configurable via `queries.extendFolder`). These will override default queries or add new ones. Example:

```graphql
# extend/queries/CustomPosts.gql
query CustomPosts($limit: Int = 5) {
  posts(first: $limit) {
    nodes {
      ...Post
      customField  # Add custom fields
    }
  }
}
```

This generates a `useCustomPosts()` composable.

### Configuration

Module configuration in `nuxt.config.ts` under the `wpNuxt` key:

```typescript
wpNuxt: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wpnuxt/wpnuxt](https://github.com/wpnuxt/wpnuxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
