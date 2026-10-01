---
trigger: always_on
description: Cross-cutting conventions that apply to every subpackage. A
---

# luxar-viewer Conventions

Cross-cutting conventions that apply to every subpackage. A
subpackage's `README.md` may override or extend these for its own
scope; the defaults below are what new code should follow unless there
is a documented reason not to.

## Table of contents

1. [File and module naming](#1-file-and-module-naming)
2. [Class and function naming](#2-class-and-function-naming)
3. [CSS class names (BEM)](#3-css-class-names-bem)
4. [Logging](#4-logging)
5. [Error handling](#5-error-handling)
6. [Resource lifecycle](#6-resource-lifecycle)
7. [Event listeners](#7-event-listeners)
8. [Result&lt;T, E&gt; for fallible operations](#8-resultt-e-for-fallible-operations)
9. [Worker safety](#9-worker-safety)
10. [Imports and barrels](#10-imports-and-barrels)
11. [Types](#11-types)
12. [Dependency inversion via ports and factories](#12-dependency-inversion-via-ports-and-factories)
13. [Error handling discipline](#13-error-handling-discipline)
14. [Resource disposal pattern](#14-resource-disposal-pattern)

---

## 1. File and module naming

- **Filenames**: kebab-case for source modules (`scene-loader.ts`,
  `material-manager.ts`). Class files match the class name in
  kebab-case (`PostProcessingManager` → `post-processing-manager.ts`).
- **Test files**: `<source-name>.test.ts` for unit tests, mirrored under
  `src/tests/unit/<area>/`. E2E tests use `.spec.ts` and live under
  `src/tests/e2e/`.
- **Index/barrel files**: only when a subpackage genuinely has a stable
  public surface (`config/index.ts`, `input/index.ts`, `rendering/index.ts`). Internal
  scratch modules import directly from each other, not through a
  barrel, to avoid cyclic imports.
- **Setup-module pattern**: when decomposing a large facade, sibling
  modules export a `setupX(context)` function returning a
  `{ controllers, folders?, cleanup? }` shape. See
  `ui/rendering-controls/types.ts` for the shared shape.

## 2. Class and function naming

- **Classes**: `UpperCamelCase`. Singletons follow the `getInstance()`
  pattern with a private constructor.
- **Functions / methods**: `lowerCamelCase`. Async methods that fetch
  remote data are named for what they return, not the verb
  (`loadScene`, not `fetchAndParseScene`).
- **Constants**: `UPPER_SNAKE_CASE` for module-level immutable values
  (`MAX_NDIM`, `TARGET_CHUNK_BYTES`). `lowerCamelCase` for everything
  else, even when "conceptually constant" (config defaults,
  pre-computed lookup tables instantiated at runtime).
- **Private members**: `private` modifier, no underscore prefix.
- **Boolean predicates**: prefix with `is` / `has` / `should` (`isDisposed`,
  `hasTransform`).

Production functions are limited to complexity 10, 120 code lines, nesting
depth 4, and 5 parameters. Existing debt is count-baselined in
`eslint-suppressions.json`; reducing a count fails lint until `pnpm lint:prune`
updates the baseline. After moving or renaming a baselined file, re-key it with
`pnpm exec eslint . --suppress-rule <rule>`, then prune and
verify the suppressions diff only moves that path.

## 3. CSS class names (BEM)

All viewer-owned class names start with `luxar-` to avoid host-page
collisions. Within that namespace, BEM applies:

```
luxar-block               // block
luxar-block__element       // element inside the block
luxar-block--modifier      // block-level modifier
luxar-block__element--state // element-level modifier
```

Examples:

- `luxar-layer-row` (block)
- `luxar-layer-row__eye` (element)
- `luxar-layer-row--selected` (modifier)
- `luxar-dimension-slider__thumb` (element)

Do **not** use Tailwind utility classes in component CSS. Themes are
CSS-variable driven (see `src/themes/README.md`); component
styles read those variables, never hard-coded values.

## 4. Logging

Channel everything through `src/utils/log.ts`:

```typescript
import { log, Modules } from '../utils/log';

log.info(Modules.RENDERER, 'Loaded scene with N points');
log.warning(Modules.CACHE, 'L2 quota exhausted, falling back to L1 only');
log.error(Modules.SCENE_LOADER, 'Failed to parse zarr metadata', err);
```

The `no-console` ESLint rule enforces this for production code. The
two carve-outs (`src/utils/log.ts` and
`src/utils/console-interceptor.ts`) are the legitimate `console.*`
sites. Tests, benchmarks, screenshot drivers, and mocks may use
`console.*` directly — they are tooling, not in-app code.

Output format is fixed: `[emoji] [Module] message`. Custom emojis go
through `log.custom(emoji, module, message)`.

## 5. Error handling

| Mechanism     | Use when                                          | Example                                           |
| ------------- | ------------------------------------------------- | ------------------------------------------------- |
| `throw`       | Unrecoverable invariant violation at JS boundary  | `validateNDArrays` rejecting a malformed buffer   |
| `Result<T,E>` | Recoverable with a typed error code               | Cache miss vs network error vs corrupt vs aborted |
| `log.warning` | Degraded behaviour, app continues                 | localStorage quota exceeded                       |
| `log.error`   | Unexpected failure, app continues but UX impacted | WebGL context lost (with rebuild scheduled)       |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [royerlab/luxar](https://github.com/royerlab/luxar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
