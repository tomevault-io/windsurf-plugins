---
trigger: always_on
description: `@tenphi/tasty` is a CSS-in-JS styling system and DSL for React. It provides declarative, state-aware styling with design token integration, sub-element styling, and zero-runtime extraction via Babel.
---

# AGENTS.md — @tenphi/tasty

## Project Overview

`@tenphi/tasty` is a CSS-in-JS styling system and DSL for React. It provides declarative, state-aware styling with design token integration, sub-element styling, and zero-runtime extraction via Babel.

Repository: <https://github.com/tenphi/tasty>

## Quick Reference

| Command | Purpose |
|---|---|
| `pnpm build` | Build via tsdown (ESM, browser + node targets) |
| `pnpm test` | Run tests (vitest) |
| `pnpm typecheck` | Type-check without emitting |
| `pnpm lint` | Lint source files |
| `pnpm lint:fix` | Lint and auto-fix |
| `pnpm format` | Format with Prettier |
| `pnpm format:check` | Check formatting |
| `pnpm bench` | Run benchmarks (vitest bench) |
| `pnpm size` | Check bundle sizes (size-limit) |
| `pnpm hygiene` | Run lint + format check + typecheck together |
| `pnpm hygiene:fix` | Auto-fix lint + format, then typecheck |

## Before pushing changes

Follow the ordered steps in [`.cursor/commands/submit-changes.md`](.cursor/commands/submit-changes.md) for the full release-oriented workflow. Before you push **any** branch, at minimum:

1. **Typecheck** — Run `pnpm typecheck`. If it fails, stop and fix errors before formatting or committing.
2. **Lint** — Run `pnpm lint`. If it fails, stop and fix errors before formatting or committing.
3. **Format** — Run `pnpm format` so committed code matches Prettier output.
4. **Changeset** — If the change affects published package behavior (features, fixes, refactors, perf), create a changeset file in `.changeset/` as described in `submit-changes.md` and include it in the commit. Use `patch` for fixes/small changes, `minor` for new features/non-breaking API changes, `major` for breaking changes. Skip the changeset only when the change is purely internal (docs, CI, repo-only churn, tests with no behavior change).
5. **Commit** — Use [Conventional Commits](https://www.conventionalcommits.org/) (`feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `ci`; optional scope). Keep the subject line short. Include the changeset file in the same commit.
6. **Push** — Do not push to `main`. Confirm the current branch, then push with `git push -u origin HEAD`.

## Stack

- **Language**: TypeScript (strict mode, `consistent-type-imports` enforced)
- **Build**: tsdown — ESM, unbundled, dts + sourcemaps, browser + node targets
- **Test**: Vitest 4, globals enabled, jsdom (default) + happy-dom (injector tests)
- **Lint**: ESLint 10 + typescript-eslint + prettier
- **Format**: Prettier — single quotes, semicolons, trailing commas, 80 cols
- **Versioning**: Changesets
- **Runtime**: Node >= 20, pnpm 10

## Entry Points

| Import path | Description | Platform |
|---|---|---|
| `@tenphi/tasty` | Runtime style engine (tasty, hooks, configure) | Browser |
| `@tenphi/tasty/core` | Core engine without SSR | Browser |
| `@tenphi/tasty/static` | Build-time static style generation (tastyStatic) | Browser |
| `@tenphi/tasty/babel-plugin` | Babel plugin for zero-runtime CSS extraction | Node |
| `@tenphi/tasty/zero` | Programmatic zero-runtime extraction API | Node |
| `@tenphi/tasty/next` | Next.js integration wrapper for zero-runtime | Node |
| `@tenphi/tasty/ssr` | Server-side rendering collector + hydration | Node |
| `@tenphi/tasty/ssr/next` | Next.js App Router SSR integration | Node |
| `@tenphi/tasty/ssr/astro` | Astro integration + middleware | Node |
| `@tenphi/tasty/ssr/astro-client` | Astro client-side cache hydration | Browser |

## Project Structure

```
src/
  index.ts              Main entry point (runtime exports)
  tasty.tsx              Core tasty() factory — creates styled React components
  config.ts              Global configuration system (configure())
  types.ts               Core TypeScript types
  debug.ts               Runtime debug/diagnostic utilities (tastyDebug)

  core/                  Core engine without SSR side-effects
  static/                tastyStatic() — build-time style generation
  zero/                  Zero-runtime CSS extraction & Babel plugin
    babel.ts             Babel plugin entry
    next.ts              Next.js wrapper
    extractor.ts         Style extraction logic
    css-writer.ts        CSS file writer

  hooks/                 React hooks
    useStyles.ts         Generate className from style definitions
    useGlobalStyles.ts   Inject global styles for a selector
    useRawCSS.ts         Inject raw CSS strings
    useKeyframes.ts      Inject @keyframes animations
    useProperty.ts       Inject CSS @property definitions

  injector/              Runtime CSS injection engine
    injector.ts          Core injector (hash dedup, ref counting, cleanup)
    sheet-manager.ts     CSSStyleSheet management
  pipeline/              Style rendering pipeline (parse → exclusives → materialize); see docs/pipeline.md
  parser/                Style value parser & tokenizer (custom DSL)
  styles/                Style property handlers (fill, padding, border, etc.)
  chunks/                Style chunking system
  states/                Predefined state mappings (@hover, @media, etc.)
  plugins/               Plugin system (OKHSL color support, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tenphi/tasty](https://github.com/tenphi/tasty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
