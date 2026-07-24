---
trigger: always_on
description: Lightweight, high-performance JavaScript/TypeScript HTTP router. Zero runtime dependencies.
---

# rou3

Lightweight, high-performance JavaScript/TypeScript HTTP router. Zero runtime dependencies.

> [!IMPORTANT]
> Keep `AGENTS.md` updated with project status.

## Project Structure

```
src/
  index.ts            # Public API re-exports
  types.ts            # TypeScript interfaces & param inference types
  context.ts          # createRouter() factory
  object.ts           # NullProtoObj (null-prototype object constructor)
  _escape.ts          # URLPattern backslash escape handling (placeholder approach)
  _group-delimiters.ts# Non-capturing group ({...}) expansion helper
  _group-names.ts     # Capture-group name codec (param name <-> group name escaping)
  _segment-wildcards.ts# Wildcard segment capture handling
  _overlap.ts         # Pattern-overlap shape model (tree entry -> RouteShape, shape intersection)
  _subsume.ts         # Shape subsumption/canonicalization (shapeSubsumes, mergeShapes, regex-key normalization)
  regexp.ts           # routeToRegExp() utility
  regexp-to-route.ts  # regExpToRoute() utility (inverse of routeToRegExp)
  compiler.ts         # JIT/AOT compiler (generates optimized match functions)
  operations/
    add.ts            # addRoute() - insert routes into the radix tree
    find.ts           # findRoute() - single-match lookup
    find-all.ts       # findAllRoutes() - multi-match lookup
    overlap.ts        # routesOverlap() / compareRoutes() / findOverlappingRoutes() - pattern-vs-pattern relations
    remove.ts         # removeRoute() - remove routes from tree
    _utils.ts         # Shared utilities (escaping, path splitting, normalization)
test/
  router.test.ts      # Core router tests
  find.test.ts        # Route matching tests (interpreter vs compiled)
  find-all.test.ts    # Multi-match tests
  overlap.test.ts     # Pattern-overlap tests (routesOverlap / compareRoutes / findOverlappingRoutes)
  group-names.test.ts # Capture-group name codec (round-trip + injectivity)
  regexp.test.ts      # RegExp conversion tests
  regexp.pcre.test.ts # Cross-engine PCRE checks (runs routeToRegExp output through installed grep -P/rg -P/pcre2grep/perl/php)
  _regexp-cases.ts    # Shared route->regex fixtures (used by regexp.test.ts + regexp.pcre.test.ts)
  types.test-d.ts     # TypeScript type-level tests
  bench/              # Performance benchmarks (mitata)
  _utils.ts           # Test helpers (createRouter, formatTree)
```

## Public API

Two entry points: `rou3` (main) and `rou3/compiler`.

```ts
// rou3
createRouter<T>(options?) -> RouterContext<T>
addRoute(ctx, method, path, data?) -> void
removeRoute(ctx, method, path) -> void
findRoute(ctx, method, path, opts?) -> MatchedRoute<T> | undefined
findAllRoutes(ctx, method, path, opts?) -> MatchedRoute<T>[]
routesOverlap(patternA, patternB) -> boolean
compareRoutes(patternA, patternB) -> "disjoint" | "equal" | "superset" | "subset" | "partial"
findOverlappingRoutes(ctx, method, pattern) -> MatchedRoute<T>[]
routeToRegExp(route) -> RegExp
regExpToRoute(regexp) -> string

// rou3/compiler
compileRouter<T>(router, opts?) -> (method, path) => MatchedRoute<T> | undefined
compileRouterToString(router, functionName?, opts?) -> string
```

## Core Algorithm

**Radix tree** with three node types: **static** (exact match), **param** (`:id`, `*`), **wildcard** (`**`).

### Node structure

```ts
interface Node<T> {
  key: string;
  static?: Record<string, Node<T>>;
  param?: Node<T>;
  wildcard?: Node<T>;
  hasRegexParam?: boolean;
  methods?: Record<string, MethodData<T>[]>;
}
```

### Lookup priority

1. Static child (exact segment match)
2. Param child (single-segment dynamic)
3. Wildcard (multi-segment catch-all)

**Same-node siblings** (multiple routes sharing one node's `methods[method]` array) resolve by one shared model in all three matchers (`_selectMatcher` in find.ts, `pushSorted` in find-all.ts, the compiled matcher): the highest specificity weight among **fully-matching** entries wins, ties go to the first-registered. Weight = one point per passing regex-constrained param + one for a required last param on a dynamic (param/wildcard) terminal. An entry whose regex fails is skipped — lookup falls through to less specific siblings, other node kinds, or the optional end-of-path fallback, never aborting (the old per-level greedy regex filter made `findRoute` miss routes that `findAllRoutes` found). Out-of-bounds `segments[i]` reads (`undefined`) must never coerce into a literal `"undefined"` static key — guard with `index < segments.length` before any `node.static[...]` lookup. Both pinned in `find.test.ts` "same-node sibling selection" and `find-all.test.ts` "out-of-bounds segment". `_selectMatcher` keeps a single-sibling/no-regex fast path — lookup perf is at parity with the pre-selection code; keep it when editing.

### `findAllRoutes` result ordering

Results are ordered **least → most specific**, and the interpreter (`findAllRoutes`) and compiled `matchAll` must agree exactly (`test/find-all.test.ts` asserts `toEqual`). This is a **public contract**, documented in README ("Result ordering") and pinned by `test/find-all.test.ts` (`matcher: ordering contract` ties result order to `compareRoutes` subsumption order) — changing it is a breaking change. Two levels:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [h3js/rou3](https://github.com/h3js/rou3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
