---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**@keenmate/svelte-spa-router** is a modern router for Svelte 5 SPAs built with runes (`$state`, `$props`, `$effect`, `$derived`). It supports dual-mode routing (hash-based `#/path` and history API `/path`) with comprehensive permission management for both role-based and resource-based access control.

**Key Technologies:**
- Svelte 5 with runes (NOT Svelte stores)
- regexparam for route pattern matching
- Vitest + Testing Library for tests
- No build step required (distributed as source)

## Critical: Import Patterns

**⚠️ IMPORTANT:** This is a Svelte 5 router using runes, NOT Svelte stores. There is **NO `/stores` export path**.

### Correct Import Patterns

```javascript
// ✅ CORRECT - Main module or /utils
import Router from '@keenmate/svelte-spa-router'
import { push, replace, location, querystring, routeParams } from '@keenmate/svelte-spa-router'
// OR
import { location, routeParams } from '@keenmate/svelte-spa-router'

// ✅ CORRECT - Use as functions (not stores!)
const path = $derived(location())        // Call as function
const params = $derived(routeParams())   // Call as function

// ✅ CORRECT - Receive as props in route components (preferred)
let { routeParams = {} } = $props()
```

```javascript
// ❌ WRONG - /stores path doesn't exist!
import { routeParams } from '@keenmate/svelte-spa-router/stores'  // ERROR!

// ❌ WRONG - Store syntax doesn't work
const path = $location  // ERROR! location is not a store
const params = $routeParams  // ERROR! routeParams is not a store

// ❌ WRONG - Old v3/v4 name
import { params } from '@keenmate/svelte-spa-router'  // Should be routeParams
```

### Common Import Errors

1. **"Missing './stores' specifier"** - Users trying to use old v3/v4 API
   - Fix: Import from main module or `/utils`, not `/stores`
   - Use `routeParams()` as a function, not `$params` as a store

2. **"params is undefined"** - Name changed in v5
   - Old: `params`
   - New: `routeParams`

3. **Event handlers not working** - Naming changed to camelCase
   - Old: `onrouteLoaded`, `onconditionsFailed`
   - New: `onRouteLoaded`, `onConditionsFailed`

### All Available Import Paths

```
@keenmate/svelte-spa-router                        // Main (Router, push, location, etc.)
@keenmate/svelte-spa-router/utils                  // Alternative for utils
@keenmate/svelte-spa-router/wrap                   // Route wrapping
@keenmate/svelte-spa-router/active                 // Active link action
@keenmate/svelte-spa-router/routes                 // Named routes
@keenmate/svelte-spa-router/helpers/permissions    // Permission system
@keenmate/svelte-spa-router/helpers/navigation-guard  // Navigation guards
@keenmate/svelte-spa-router/helpers/hierarchy      // Hierarchical routes
@keenmate/svelte-spa-router/helpers/error-handler  // Error handling
@keenmate/svelte-spa-router/helpers/*              // Other helpers
```

**NO `/stores` path exists - this router uses functions, not stores!**

## Common Commands

```bash
# Testing
npm test              # Run all tests
npm run test:watch    # Run tests in watch mode
npm run test:ui       # Run tests with UI
npm run test:coverage # Run tests with coverage report

# Development (Examples)
make dev              # Run history mode example (clean URLs)
make dev-hash         # Run hash mode example (#/path URLs)

# Linting
npm run lint          # Run ESLint

# Building examples
make build-examples   # Build both example apps
make build-hash       # Build hash mode example only
make build-history    # Build history mode example only
```

**Note:** There is no build step for the library itself. The package is distributed as source files.

## Architecture Overview

### Core Module Structure

The router is organized into several key modules:

**Router.svelte** - Main router component
- Uses `$effect()` to watch location changes and match routes
- Handles async component loading with race condition protection
- Manages route conditions/guards evaluation
- Implements scroll restoration with browser History API
- Event system via callback props (onRouteLoading, onRouteLoaded, onConditionsFailed, onNotFound)

**utils.svelte.js** - Core routing utilities and state management
- Contains all reactive state using `$state()` (locationState, paramsState, navigationContextState)
- Dual-mode routing: hash-based (default) or history API
- Configuration: `setHashRoutingEnabled()`, `setBasePath()`, `setParamReplacementPlaceholder()`
- Navigation functions: `push()`, `pop()`, `replace()`, `goBack()` with multi-parameter signatures
- `goBack()` - Navigate to referrer with automatic scroll position restoration (requires referrer tracking)
- State accessors: `location()`, `querystring()`, `routeParams()`, `navigationContext()`, `loc()`
- `link` action for SPA navigation with modifier key support and 4-element array format

**wrap.js** - Route wrapping utility
- Enables async component loading and code splitting
- Supports loading components while routes load
- Adds route conditions/guards
- Attaches static props and user data to routes

**active.svelte.js** - Active link highlighting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Keenmate/svelte-spa-router](https://github.com/Keenmate/svelte-spa-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
