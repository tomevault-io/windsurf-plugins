---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Essential Commands

### Development

```bash
bun run dev               # Start development with watch mode
bun run build             # Build for production (outputs to dist/)
bun ./dist/browser/index.js  # Run the browser-safe ESM build
node ./dist/browser/index.js # Run the browser-safe ESM build
```

### Testing

```bash
bun test                  # Run all tests
bun test src/utils        # Run tests in specific directory
bun test logger           # Run tests matching pattern
bun test --watch          # Watch mode
bun test --coverage       # Generate coverage report
```

### Code Quality

```bash
bun run lint             # Check linting errors
bun run lint:fix         # Auto-fix linting errors
bun run typecheck        # TypeScript type checking (src + scripts)
bun run typecheck:test   # TypeScript type checking (test files)
bun run format           # Format all files with Prettier
bun run format:check     # Check formatting without changes
bun run check            # Fast local sanity: format:check + lint + typecheck
bun run validate         # Full gate: format:check + lint + typecheck + typecheck:test + test + build + package:check
```

### Utilities

```bash
bun run clean            # Clean build artifacts (dist/, coverage/, caches)
bun run package:check    # Run publint + @arethetypeswrong/cli on packed tarball
```

## Architecture Overview

### Core Design Principles

1. **Immutable value objects**: `Note`, `Chord`, and `Scale` are the primary types. All fields are
   private `#fields` (non-writable by spec). No `Object.freeze` wrappers.

2. **Branded types for domain validation**: `MidiKey`, `Frequency`, `Semitones`, `Octave`, and
   `ChromaticIndex` are `Brand<number, …>` types validated at construction time. Public method
   parameters accept plain `number`; brands are applied internally via the `create*` helpers.

3. **Catalog-driven data with alias resolution**: `INTERVALS`, `CHORDS`, and `SCALES` are the
   authoritative data catalogs. Every alias resolves to a canonical key via `resolveInterval`,
   `resolveChordSuffix`, and `resolveScaleType`.

4. **Runtime-neutral published code**: `src/` must not use Bun-only runtime APIs (`Bun.file`,
   `Bun.env`, `Bun.serve`, etc.). Those APIs are fine in `scripts/` and test files, but must not
   appear in published library output.

### Key Notes

- **ESM-only**: The package is ESM-only. Node 22+ resolves via the `"import"` condition. There is no
  CJS bundle.
- **ESM + TypeScript**: Source files are TypeScript modules; build output targets a browser-safe ESM
  bundle.
- **Import paths**: Use standard TS/ESM imports; no `@/*` path alias (it leaks into `.d.ts` files).
- **Library output**: Single browser-safe ESM bundle in `dist/browser/` for all runtimes (Node 22+,
  Bun, browser bundlers). The `exports` map routes consumers automatically.

### Library Packaging

The build produces:

- `dist/browser/index.js` — ESM bundle, `Bun.build target: 'browser'`, no external deps (zero
  runtime dependencies)
- `dist/index.d.ts` — TypeScript declarations (shared, generated with `isolatedDeclarations: true`)

The `exports` map in `package.json`:

```json
{
  ".": {
    "types": "./dist/index.d.ts",
    "browser": "./dist/browser/index.js",
    "import": "./dist/browser/index.js",
    "default": "./dist/browser/index.js"
  },
  "./package.json": "./package.json"
}
```

Package validation runs as part of `validate`: `publint` checks the exports map structure and
`@arethetypeswrong/cli` checks type resolution across resolution modes.

### Git Hooks Architecture

Hooks are configured in `lefthook.yml` and implemented as Bun TypeScript files under
`scripts/hooks/`:

- **pre-commit** (`lefthook.yml` inline, piped/sequential): formats staged files with Prettier, runs
  oxlint --fix on staged files, checks `bun.lock` is staged when `package.json` changes. Fast by
  design.
- **pre-push** (`lefthook.yml`): runs format check, lint, typecheck, and tests in parallel (build
  and package check are CI-only).
- **post-checkout** (`scripts/hooks/post-checkout.ts`): installs deps when `package.json`+`bun.lock`
  change; surfaces config changes.
- **post-merge** (`scripts/hooks/post-merge.ts`): installs/cleans when dependencies or config
  changed; shows merge stats.

They use `chalk` for color and Bun's `$` and `Bun.write` for shell/IO.

### Types

There is no shared `src/types.ts`. Domain-specific types live near their modules (e.g.,
`SerializedNote` in `note.ts`, `SerializedChord` in `chord.ts`).

## Development Patterns

### Adding New Features

1. **Catalog entries**: New intervals, chord suffixes, or scale types go into the corresponding
   catalog file (`src/intervals.ts`, `src/chords.ts`, `src/scales.ts`). Add a canonical entry first,
   then any aliases.
2. **Types**: Domain-specific types live near their modules.

### Testing Approach

- Tests use Bun's built-in test runner with `describe`, `it`, `expect`.
- Test files are colocated with sources using the `.test.ts` suffix.
- `test/setup.ts` is preloaded by `bunfig.toml` — it resets mocks and system time in `afterEach`.
  All tests get this automatically.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevekinney/octavian](https://github.com/stevekinney/octavian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
