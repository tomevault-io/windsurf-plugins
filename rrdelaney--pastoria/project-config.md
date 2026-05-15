---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Pastoria is a full-stack JavaScript framework for building data-driven
applications. This repository contains only the framework itself - not an
application built with it. The framework combines:

- **File-based routing** with type-safe navigation and Zod param validation
- **React Relay** for GraphQL data fetching with persisted queries and
  entrypoint-based preloading
- **Vite** for build tooling and dev server
- **Server-side rendering (SSR)** with React
- **TailwindCSS** for styling
- **Code generation** that wires routes, queries, and entrypoints together
  automatically

## Monorepo Structure

This is a pnpm workspace monorepo with the following packages:

- **`packages/pastoria`**: Main CLI tool that provides `generate`, `dev`, and
  `build` commands for framework users
- **`packages/runtime`**: Runtime utilities for routing, Relay environments, and
  server handlers
- **`packages/server`**: Standalone production server for framework users
- **`packages/pastoria`**: Scaffolding tool for creating new Pastoria projects

The `website/` directory contains a Docusaurus docs site but is outdated and
should be ignored.

## Common Commands

### Building the Framework

```bash
# Build the framework packages
pnpm run --filter './packages/*' build

# Type check all packages
pnpm check:types
```

### Formatting

```bash
# Format code
pnpm format

# Check formatting
pnpm check:format
```

### Publishing

```bash
# Create a changeset
pnpm changeset

# Version packages
pnpm changeset version

# Install to update lockfile
pnpm install

# Build all packages
pnpm run -r build

# Publish to npm
pnpm publish -r
```

## What a Pastoria App Looks Like

### App Directory Structure

A Pastoria app has this layout:

```
my-app/
  pastoria/                          # Framework source directory
    app.tsx                          # App shell (wraps all pages)
    environment.ts                   # PastoriaEnvironment config (GraphQL schema)
    globals.css                      # Global styles (imports tailwindcss)
    page.tsx                         # Route: /
    about/page.tsx                   # Route: /about
    hello/[name]/
      page.tsx                       # Route: /hello/[name]
      banner.tsx                     # Nested entrypoint: /hello/[name]#banner
      results.tsx                    # Nested entrypoint: /hello/[name]#results
    api/greet/[name]/
      route.ts                       # Server API route: /api/greet/:name
  src/                               # Shared app code, schema resolvers, etc.
  __generated__/                     # All generated code (never hand-edit)
    router/                          # Pastoria-generated router artifacts
    queries/                         # Relay-generated query artifacts
    schema/                          # grats-generated GraphQL schema
  package.json
  tsconfig.json
  relay.config.json
```

### Path Aliases

Apps use Node.js subpath imports (configured in both `package.json` `"imports"`
and `tsconfig.json` `"paths"`). Two are required by the framework:

- `#pastoria/*` → `./pastoria/*`
- `#genfiles/*` → `./__generated__/*`

Apps conventionally also add `#src/*` → `./src/*` for their own code, but this
is not required by the framework.

### App Configuration

**`pastoria/environment.ts`** — exports a `PastoriaEnvironment` instance:

```ts
import {PastoriaEnvironment} from '@pastoria/runtime/server';
export default new PastoriaEnvironment({
  schema, // GraphQLSchema (required)
  createContext: (req) => new Context(), // per-request context factory
  enableGraphiQLInProduction: false,
  persistedQueriesOnlyInProduction: true,
});
```

**`pastoria/app.tsx`** — the app shell component wrapping all pages. If present,
replaces the default HTML shell:

```tsx
import type {PropsWithChildren} from 'react';
import './globals.css';

export default function AppRoot({children}: PropsWithChildren) {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <title>My App</title>
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### Package.json Scripts

```json
{
  "scripts": {
    "generate": "grats && relay-compiler && pastoria generate",
    "build": "pastoria build",
    "dev": "pastoria dev",
    "start": "NODE_ENV=production pastoria-server"
  }
}
```

The generate pipeline is a three-step sequence: `grats` (GraphQL schema from TS
JSDoc) → `relay-compiler` (Relay query artifacts) → `pastoria generate` (router,
entrypoints, types).

## File-Based Routing

### Route Conventions

- **`page.tsx`** — any file named `page.tsx` under `pastoria/` becomes a
  navigable route. The directory path maps to the URL. Dynamic segments use
  `[param]` notation:
  - `pastoria/hello/[name]/page.tsx` → route `/hello/[name]`
  - `pastoria/search/page.tsx` → route `/search`
- **Non-page `.tsx` files** — any other `.tsx` file under `pastoria/` (excluding
  `app.tsx` and `environment.ts`) becomes a **nested entrypoint** (lazy
  sub-component). Its ID uses `#` to separate path from filename:
  - `pastoria/hello/[name]/banner.tsx` → `/hello/[name]#banner`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rrdelaney/pastoria](https://github.com/rrdelaney/pastoria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
