---
trigger: always_on
description: The @svelte-router/core routing library supports simultaneous path and hash routing through "routing universes":
---

# Svelte Router Routing Library

## Library Architecture Overview

### Routing Universes Concept

The @svelte-router/core routing library supports simultaneous path and hash routing through "routing universes":

- **Path Routing** (`hash: false`): Uses URL pathname
- **Single Hash Routing** (`hash: true`): Uses URL hash as a single path (e.g., `#/path/to/route`)
- **Multi Hash Routing** (`hash: 'p1'`): Uses semicolon-separated hash segments (e.g., `#p1=/path;p2=/other`)
- **Implicit Path Routing** (`hash: undefined`, `defaultHash: false`): Resolves to path routing
- **Implicit Hash Routing** (`hash: undefined`, `defaultHash: true`): Resolves to hash routing
- **Implicit Named Hash Routing** (`hash: undefined`, `defaultHash: <a string>`): Resolves to named (multi) hash routing

#### Example Multi-Universe Setup

```svelte
<Router>
    <Route path="/hash-feature/*">
        <Router hash>
            <Route hash path="/">
                <RouteHashHome />
            </Route>
            <Route path="/non-hash-route">
                <NonHashContentForSomeReason />
            </Route>
        </Router>
    </Route>
</Router>
```

### Context System

Context is stored per universe using Svelte's `setContext()` with keys from `getRouterContextKey()`:

- Path routing: `parentCtxKey` symbol
- Single hash routing: `hashParentCtxKey` symbol
- Multi hash routing: `Symbol.for('hsh-${hashId}')` per hash ID

### RouterEngine Core

The `RouterEngine` class is the heart of the routing system:

#### Routes Structure

```typescript
routes: Record<string, RouteInfo>;
```

Where `RouteInfo` contains:

- `pattern?: string` or `regex?: RegExp`: For URL matching
- `and?: (routeParams) => boolean`: Additional predicate for additional constraining, like guarded routes
- `ignoreForFallback?: boolean`: Excludes route from fallback calculations

#### Reactive Properties

- `routeStatus`: Per-route match status and extracted parameters
- `fallback`: Boolean indicating NO routes matched (excluding `ignoreForFallback` routes)

### Component Architecture

Except for the `LinkContext` components, all components possess the `hash: Hash` property to specify the routing
universe they belong to.

#### Router Component

- Creates `RouterEngine` instance
- Sets up context for child components
- Provides `state` and `routeStatus` to children

#### Route Component

- Registers route patterns with parent router
- Uses context to find parent router
- Props: `path` (string pattern/regex) and `and` (predicate function)

#### Fallback Component

- Shows content when no routes match
- Props:
    - `when?: WhenPredicate`: Override default `fallback` behavior
    - `children`: Content snippet

Render logic:

```svelte
{#if (router && when?.(router.routeStatus, router.fallback)) || (!when && router?.fallback)}
    {@render children?.(router.state, router.routeStatus)}
{/if}
```

## Library Initialization

The `init()` function:

- Passes library configuration to global singleton
- Creates new Location implementation instance
- Returns a cleanup function; mainly used in unit testing but valid anywhere as required.
- Required when library configurations change
- Multi hash routing needs cleanup between tests

```typescript
// Standard init
const cleanup = init();

// Multi hash mode
const cleanup = init({ hashMode: 'multi' });

// Always cleanup
afterAll(() => {
    cleanup();
});
```

## Extensibility

This library supports the creation of extension NPM packages by allowing custom implementations of the `Location`, `HistoryApi` and `FullModeHistoryApi` interfaces.

`Location` implementations are in charge of obtaining the current environment URL and keeping it in sync across navigation. They read the environment's URL and sets up reactive `$state` and `$derived` data for the rest of components and the application that consumes the package.

`HistoryApi` implementations are helpers for the `Location` implementations. They either tap into or completely replace the environment's history API to fulfill the sought objective.

NPM extension packages may opt to provide custom implementations for any of these interfaces. They may (and are encouraged to) expose their own initialization functions to provide customized or new options, and to remove stock initialization options that should not be touched by consumers.

Custom initialization functions must ultimately call this package's `initCore()` function with the desired values; NPM extension packages can provide any number of initialization functions.

### Location Implementations

#### LocationLite

- Default library option
- By default, uses the `StockHistoryApi` class as its `HistoryApi` implementation
- Base class for `LocationFull`

#### LocationFull

- By default, uses the `InterceptedHistoryApi` class as its `FullModeHistoryApi` implementation

### HistoryApi and FullModeHistoryApi Implementations

#### StockHistoryApi

- Relays all functionality to the environment's history object
- Synchronizes reactive values as needed while being used

#### InterceptedHistoryApi

- Extends the `StockHistoryApi` class
- Replaces the environment's history object with itself on construction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WJSoftware/svelte-router](https://github.com/WJSoftware/svelte-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
