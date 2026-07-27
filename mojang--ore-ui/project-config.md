---
trigger: always_on
description: > **Note**: This is the comprehensive internal guide for contributors. For a streamlined public API reference, see [`copilot-instructions-public-api.md`](./copilot-instructions-public-api.md).
---

# GitHub Copilot Instructions for Ore UI / React Facet

> **Note**: This is the comprehensive internal guide for contributors. For a streamlined public API reference, see [`copilot-instructions-public-api.md`](./copilot-instructions-public-api.md).
>
> **Maintenance**: When updating this file, also update the public API version if changes affect public APIs, usage patterns, or best practices. Run `./scripts/check-copilot-instructions-sync.sh` and `./scripts/check-public-api-instructions-sync.sh` to validate both files.

## Project Overview

**Ore UI** is Mojang Studios' open-source collection of building blocks for constructing video game user interfaces using web standards. The flagship package is **React Facet** (`@react-facet`), an observable-based state management system designed for performant game UIs built in React.

### Target Use Case

- **Primary**: Game UI development using embedded web technologies (Coherent Labs' Gameface)
- **Games using this**: Minecraft Bedrock Edition, Minecraft Legends
- **Performance Requirements**: Fixed frame budget, optimized for slower devices

### Core Philosophy

React Facet bypasses React reconciliation for leaf node updates (styles, text content, attributes) to achieve game-level performance while maintaining React's developer experience.

---

## ⚠️ Top 3 Critical Errors to Avoid

Before diving into the details, be aware of these critical mistakes that **defeat the entire purpose of React Facet**:

### 1. 🚨 CRITICAL: Forgetting to Check for NO_VALUE

**Problem**: `useFacetUnwrap` and setter callbacks return `T | NO_VALUE`, not just `T`. Using the value without checking causes TypeScript errors and runtime bugs.

```typescript
// ❌ WRONG - TypeScript ERROR!
const value = useFacetUnwrap(numberFacet)
const doubled = value * 2 // Error: NO_VALUE is not a number

const [items, setItems] = useFacetState<string[]>([])
setItems((current) => [...current, 'new']) // Error: NO_VALUE is not spreadable

// ✅ CORRECT - Always check for NO_VALUE
const value = useFacetUnwrap(numberFacet)
if (value !== NO_VALUE) {
  const doubled = value * 2 // ✓ Safe
}

setItems((current) => (current !== NO_VALUE ? [...current, 'new'] : ['new']))
```

**Remember**:

- `useFacetUnwrap` → always returns `T | NO_VALUE`
- Setter callbacks → always receive `T | NO_VALUE`
- Check `!== NO_VALUE` before using the value

### 2. 🚨 CRITICAL: Overusing useFacetUnwrap

**Problem**: `useFacetUnwrap` causes React re-renders, defeating the entire performance benefit of facets.

```typescript
// ❌ WRONG - Causes re-renders, defeats facet purpose!
const value = useFacetUnwrap(facet)
return <div>{value}</div>

// ✅ CORRECT - Use fast-text, no re-renders
return <fast-text text={facet} />

// ❌ WRONG - Unwrapping for conditional rendering
const isVisible = useFacetUnwrap(isVisibleFacet)
if (isVisible !== NO_VALUE && !isVisible) return null

// ✅ CORRECT - Use Mount component
<Mount when={isVisibleFacet}>
  <ExpensiveComponent />
</Mount>
```

**Rule**: Only use `useFacetUnwrap` as a **last resort** when interfacing with non-facet-aware third-party components. Otherwise, use `fast-*` components or facet-aware patterns.

### 3. 🚨 CRITICAL: Missing Dependencies in First Array

**Problem**: Facet hooks have TWO dependency arrays. Forgetting non-facet dependencies in the first array causes stale closures.

```typescript
// ❌ WRONG - Missing multiplier in first array
const multiplier = props.multiplier
const result = useFacetMap(
  (value) => value * multiplier,
  [], // ❌ Missing: [multiplier] - will use stale value!
  [valueFacet],
)

// ✅ CORRECT - Include all non-facet dependencies
const result = useFacetMap(
  (value) => value * multiplier,
  [multiplier], // ✅ Non-facet dependencies here
  [valueFacet], // ✅ Facet dependencies here
)
```

**Rule**: First array = non-facet deps (props, local vars, functions). Second array = facet deps.

---

## Repository Structure

This is a **yarn workspace monorepo** with the following organization:

### Package Structure (`packages/@react-facet/`)

```
packages/@react-facet/
├── core/                           # Core facet implementation
│   └── src/
│       ├── facet/                  # createFacet, createStaticFacet, createReadOnlyFacet
│       ├── hooks/                  # All useFacet* hooks
│       ├── components/             # Map, Mount, With
│       ├── mapFacets/              # Facet composition utilities
│       ├── equalityChecks.ts       # Equality check functions
│       ├── createFacetContext.tsx  # Context utilities
│       └── types.ts                # Core type definitions
│
├── dom-fiber/                      # Custom React renderer
│   └── src/
│       ├── fast-* components       # Facet-native DOM elements
│       └── renderer implementation
│
├── dom-fiber-testing-library/      # Testing utilities
│   └── src/
│       └── render, act utilities
│
└── shared-facet/                   # Gameface integration
    └── src/
        └── useSharedFacet, Context
```

### Examples & Documentation

```
examples/
└── benchmarking/                   # Performance benchmarks and examples

docs/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mojang/ore-ui](https://github.com/Mojang/ore-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
