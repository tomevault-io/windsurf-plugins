---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Turborepo monorepo demonstrating Micro-Frontend (MFE) architecture with Module Federation and shared state management using RTK Query.

**Tech Stack:** React 18, TypeScript 5.4, Vite 5.4, Redux Toolkit 2.2, TailwindCSS 3.4, Turbo 2.3

## Commands

```bash
# Development - starts all apps and mock API (15 concurrent processes)
yarn dev

# Demo apps (for testing URL-based invalidation)
yarn demo:dev

# Build all packages/apps
yarn build

# Lint all projects
yarn lint

# Format code
yarn format

# Full cleanup
yarn clean
```

**Per-app standalone mode** (run MFE in isolation without federation):
```bash
cd apps/mfe-profile && yarn dev:standalone
```

## Architecture

### Monorepo Structure

```
apps/
  host/              # Shell app (port 3000) - orchestrates MFEs via Module Federation
  mfe-admin/         # Admin MFE (port 3004)
  mfe-profile/       # Profile MFE (port 3002)
  mfe-orders/        # Orders MFE (port 3003)
  remote-app/        # Demo app (port 3001)
  mock-api/          # Express REST API (port 4000)
  demo-list-add/     # Demo: List + Add item (tests POST invalidation)
  demo-list-delete/  # Demo: List + Delete item (tests DELETE invalidation)
  demo-list-update/  # Demo: List + Update item (tests PUT/PATCH invalidation)
  demo-cross-resource/ # Demo: Cross-resource invalidation (Users MFE → Orders MFE, port 4006)
  demo-tag-based/    # Demo: Tag-based invalidation with fine-grained control (port 4007)

packages/
  dtsl-rtk-query/    # @dtsl/rtk-query - RTK Query wrapper for MFEs (core library) ⭐
  shared-api/        # @repo/shared-api - API layer wrapper
  ui/                # @repo/ui - Shared React components
  utils/             # @repo/utils - Utility functions
  tsconfig/          # @repo/tsconfig - Shared TS configs
```

### Module Federation Flow

Host app consumes remote MFEs via `@originjs/vite-plugin-federation`:
- Remotes expose components via `remoteEntry.js` files
- Shared dependencies (react, react-dom, react-redux, @reduxjs/toolkit) are deduplicated
- MFEs can run standalone for development or as federated remotes

## Core Package: @dtsl/rtk-query

> **For comprehensive documentation, see: `packages/dtsl-rtk-query/AGENTS.md`**

This is the central innovation of this demo - a drop-in replacement for Redux Toolkit Query with transparent MFE support.

### Key Features

| Feature | Description |
|---------|-------------|
| **Request Coalescing** | Multiple MFEs requesting same data = 1 API call |
| **Shared Cache** | Data loaded by one MFE is instantly available to others |
| **Unified Invalidation** | Mutation in one MFE updates views across ALL MFEs |
| **Reference Counting** | Prevents premature cache eviction |
| **URL-Based Auto-Invalidation** | Automatic cache invalidation based on REST URL patterns - NO TAGS NEEDED |

### URL-Based Auto-Invalidation (Zero-Config)

Eliminates the need for cross-MFE tag coordination:

```
Mutation                    Automatically Invalidates
─────────────────────────   ─────────────────────────
POST   /users         ───►  GET /users (list queries)
PUT    /users/123     ───►  GET /users + GET /users/123
PATCH  /users/123     ───►  GET /users + GET /users/123
DELETE /users/123     ───►  GET /users + GET /users/123
```

Cross-resource invalidation for complex relationships:
```typescript
import { urlInvalidationManager } from '@dtsl/rtk-query';

// When any user is modified, also invalidate orders
urlInvalidationManager.invalidateOn('/users/*', ['/orders', '/orders/*']);
```

### Quick Start

```typescript
// Replace imports - that's it!
- import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
+ import { createApi, fetchBaseQuery } from '@dtsl/rtk-query/react';
```

Key exports: `createApi`, `fetchBaseQuery`, `Provider`, `urlInvalidationManager`

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                           @dtsl/rtk-query Architecture                        │
├──────────────────────────────────────────────────────────────────────────────┤
│  APPLICATION LAYER: MFE-Profile, MFE-Orders, MFE-Admin                       │
│           │                  │                  │                            │
│           ▼                  ▼                  ▼                            │
│  INTEGRATION LAYER: createApi (Enhanced) → Provider (MFE-aware)              │
│           │                                                                  │
│           ▼                                                                  │
│  MIDDLEWARE LAYER: Cache Lifecycle + URL Invalidation Middleware             │
│           │                                                                  │
│           ▼                                                                  │
│  CORE LAYER: Global Registry | MFE Context | RefCount | Request Tracker      │
│              URL Invalidation Manager (pattern matching)                     │
│           │                                                                  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/esanmohammad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
