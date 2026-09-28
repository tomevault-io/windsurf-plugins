---
trigger: always_on
description: `obsidian-test-mocks` is a standalone npm package providing comprehensive test mocks for the Obsidian plugin API. It publishes as a dual-format (ESM + CJS) package with seven entry points: `obsidian`, `setup`, `vitest-setup`, `jest-setup`, and the three deprecated no-ops `obsidian-typings/setup`, `obsidian-typings/vitest-setup`, and `obsidian-typings/jest-setup`.
---

# AGENTS.md

## Project Overview

`obsidian-test-mocks` is a standalone npm package providing comprehensive test mocks for the Obsidian plugin API. It publishes as a dual-format (ESM + CJS) package with seven entry points: `obsidian`, `setup`, `vitest-setup`, `jest-setup`, and the three deprecated no-ops `obsidian-typings/setup`, `obsidian-typings/vitest-setup`, and `obsidian-typings/jest-setup`.

## Commands

- `npm test` — run tests (Vitest)
- `npm run test:coverage` — run tests with v8 coverage
- `npm run test:watch` — watch mode
- `npm run lint` — run ESLint
- `npm run lint:fix` — auto-fix lint issues
- `npm run format` — format with dprint
- `npm run format:check` — check formatting
- `npm run lint:md` — lint markdown files
- `npm run lint:md:fix` — auto-fix markdown lint issues
- `npm run spellcheck` — spell check with cspell
- `npm run build` — full build pipeline
- `npm run build:compile` — TypeScript type-check only
- `npm run version` — run build (used as npm version hook)
- `npm run docs:build` — generate the API reference + OG images, build the Astro site, then link-check it
- `npm run docs:dev` — regenerate the API reference, then run the Astro dev server
- `npm run docs:preview` — serve the already-built `docs/dist`

## Architecture

### Directory Structure

- `src/obsidian/` — mocks for every class/function in `obsidian.d.ts`
- `src/obsidian-typings/` — deprecated no-op setup entry points, kept for one major so the ~30 consumer repos that name them in a Vitest/Jest config do not fail at runner startup. The bridge layer they used to install is gone: the mocks carry Obsidian's real internal names themselves (L4).
- `src/globals/` — prototype extensions Obsidian adds to DOM/JS builtins (HTMLElement, Document, Array, String, etc.)
- `src/internal/` — shared implementation details NOT exported from the package
- `docs/` — the Astro + Starlight documentation site (`docs/src` is its `srcDir`; `docs/dist` the build
  output; `docs/public` its static assets). See [Documentation site](#documentation-site).
- `scripts/docs-gen/` — the ts-morph API-reference generator and the satori OG-image generator that feed it

### Key Design Decisions

L1. **Only expose what Obsidian actually has.** The package must mock exactly the public API — no extra classes, no internal helpers in the public surface. Anything Obsidian does not have belongs in `src/internal/`. "What Obsidian has" is `obsidian.d.ts` PLUS the internals `obsidian-typings` declares: a member such as `Menu.items` is real, merely undeclared publicly, so a mock may implement it (see L4). Only the package's EXPORTED classes stay strictly `obsidian.d.ts`-bound — an `obsidian-typings` type with no `obsidian.d.ts` counterpart (`Plugins`, `Commands`) never becomes a `src/obsidian/` export. It may still be implemented in `src/internal/`, which is exactly what L7 already does for the `DataAdapter` interface: `Plugins` lives in `src/internal/plugins.ts` and `App.plugins` points at it. That placement is invisible to the public surface yet fully typed for consumers, because `src/internal/` is emitted into `dist` and referenced by relative path from the public declarations (`FileSystemAdapter.d.mts` already imports `InMemoryAdapter` that way).

L2. **Meaningful implementations first.** Mocks should have real in-memory behavior (state tracking, callback invocation, data storage). Only use `noop()` (sync) or `await noopAsync()` (async) from `src/internal/noop.ts` for methods whose bodies would otherwise be completely empty (pure UI operations with no meaningful implementation, e.g., rendering, focus). If a method already has any logic in its body, do not add `noop()` or `await noopAsync()` — they are only for otherwise-empty methods.

L3. **No `obsidian-typings` imports in `src/`.** The `obsidian-typings` package uses `declare module 'obsidian'` augmentation which activates globally on import. To avoid side effects, all needed type shapes are inlined in `src/internal/types.ts`. It is a **devDependency only**, so the no-runtime-dependency guarantee holds. The one place that reads it is `scripts/helpers/obsidian-typings-surface.ts`, which builds its **own** throwaway `Program` for the conformance tests — the augmentation is visible inside that program and nowhere else, which is precisely why it does not violate this rule.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mnaoumov/obsidian-test-mocks](https://github.com/mnaoumov/obsidian-test-mocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
