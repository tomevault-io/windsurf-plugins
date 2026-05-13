---
trigger: always_on
description: pnpm build          # Build with tsdown (ESM + CJS + types)
---

# CLAUDE.md

## Commands

```bash
pnpm build          # Build with tsdown (ESM + CJS + types)
pnpm test           # Run all tests (vitest run)
pnpm test:watch     # Run tests in watch mode
pnpm lint           # Lint with oxlint
pnpm format         # Format with oxfmt
pnpm format:check   # Check formatting
pnpm typecheck      # Type check with tsgo (TypeScript native compiler)
```

Run a single test file: `pnpm vitest run tests/rules/no-duplicate-classes.test.ts`

## Versioning

Always use **semver** for version bumps: patch (x.y.Z) for bugfixes only, minor (x.Y.0) for new features or non-breaking additions, major (X.0.0) for breaking changes.

## Architecture

oxlint plugin with 22 Tailwind CSS v4 linting rules. Uses `@oxlint/plugins`' `createOnce` API (runs once per lint session; returned visitors run on every matching AST node).

Core sync/async bridge: `@tailwindcss/node`'s `__unstable__loadDesignSystem` is async, but `createOnce` is sync. Two strategies:

1. **Precompute** (`sync-loader.ts`): `execFileSync` child process pre-computes validity, canonical forms, CSS props, etc. as JSON. Runs ONCE per unique CSS entry point, cached on disk via two-level cache (mtime index + content hash). Content-based caching allows monorepo packages with identical CSS to share a single cache entry.
2. **Sort service** (`sort-service.ts`): Worker thread communicates via `SharedArrayBuffer` + `Atomics.wait()` for `enforce-sort-order`. Loads the DS once, then accepts sort requests synchronously with built-in timeout support. This calls `ds.getClassOrder()` dynamically with the actual classes, producing the exact official Tailwind sort order (identical to oxfmt/prettier-plugin-tailwindcss). The parent thread resolves `@tailwindcss/node` via `require.resolve()` and passes the path to the worker — this is critical for VS Code's extension host where module resolution context differs. Falls back to heuristic sort if the worker fails to initialize.
3. **Canonicalize service** (`canonicalize-service.ts`): Worker thread (same SharedArrayBuffer + Atomics pattern as sort-service) for `enforce-canonical`. Calls `ds.canonicalizeCandidates([cls], { rem })` one class at a time (the API deduplicates input, so batching loses order/length for inputs with duplicates). Enables canonicalization of arbitrary user classes (`p-[2px]` → `p-0.5`, `text-[var(--x)]/90` → `text-(--x)/90`, `theme()` functions, etc.) that can't be precomputed. `rem` comes from `settings.tailwindcss.rootFontSize` (default: 16). Has a process-wide per-class cache keyed by `${cssPath}\0${rem}\0${class}` — only cache misses cross the worker boundary. Falls back to precomputed cache if worker fails. `enforce-canonical` itself only routes classes with `[` or `(` in the utility (detected via `utilityHasDynamicValue`) to the worker; named classes resolve directly via `cache.canonicalize` (sync, sub-microsecond, already preserves `!` position), which is ~5x faster in practice.

DS-dependent rules: `no-unknown-classes`, `no-conflicting-classes`, `no-deprecated-classes`, `enforce-canonical`, `enforce-sort-order`, `no-unnecessary-arbitrary-value`. `consistent-variant-order` optionally uses DS.

## Extraction System

`extractors.ts` is the shared class-detection layer used by all 22 rules. Every rule delegates to `createExtractorVisitors(context, check)` which generates the 4 standard AST visitors and resolves the extractor config lazily from `settings.tailwindcss`.

**Default detection targets** (extended additively via settings):

- **Attributes**: `className`, `class` (JSX)
- **Callees** (14): `cn`, `clsx`, `cva`, `twMerge`, `tv`, `cx`, `classnames`, `ctl`, `twJoin`, `cc`, `clb`, `cnb`, `objstr`, `classed`
- **Tags**: `tw` (tagged template literals: `` tw`bg-red-500` ``)
- **Variable patterns**: identifiers matching `/^classNames?$/`, `/^classes$/`, `/^styles?$/`

**Custom configuration** via `settings.tailwindcss` (all additive to defaults):

- `attributes: string[]` — additional JSX attribute names (e.g. `["xyzClassName", "classNames"]`)
- `callees: string[]` — additional function names (e.g. `["myHelper"]`)
- `tags: string[]` — additional tagged template tags
- `variablePatterns: string[]` — additional regex patterns for variable names (as strings, compiled to RegExp)
- `exclude: { attributes?, callees?, tags?, variablePatterns? }` — remove specific items from defaults. For `variablePatterns`, exclusions match against `RegExp.source` (e.g. `"^styles?$"` removes `/^styles?$/`).

Config is resolved lazily by `getExtractorConfig(context)` on first visitor call (settings unavailable in `createOnce`). Cached in module-level variable; `resetExtractorConfig()` for test isolation.

**Deep extraction**: `cva()` understands `variants`, `compoundVariants`, ignores `defaultVariants`. `tv()` understands `base`, `slots`, `variants` (with slot sub-objects), `compoundVariants`, `compoundSlots`. `classed()` (tw-classed) skips first arg (element type), then extracts class strings and cva-like config from remaining args.

- **JSX object values**: `classNames={{ root: "flex", label: "text-sm" }}` extracts string values from the object (not keys). This is distinct from call-expression objects like `cn({ "bg-red-500": cond })` which extract keys.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergioazoc/oxlint-tailwindcss](https://github.com/sergioazoc/oxlint-tailwindcss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
