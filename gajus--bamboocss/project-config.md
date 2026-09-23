---
trigger: always_on
description: This guide helps AI assistants understand the Bamboo CSS codebase structure, conventions, and best practices.
---

# Claude Code Guide for Bamboo CSS

This guide helps AI assistants understand the Bamboo CSS codebase structure, conventions, and best practices.

## Project Overview

Bamboo CSS is a CSS-in-JS framework with static extraction capabilities. The project is a monorepo managed by **pnpm**
with workspace support.

## Key Architecture

### Monorepo Structure

```
/packages/          # Core packages published to npm
  /core/           # CSS processing, rule generation, optimization
  /shared/         # Runtime helpers shipped into styled-system (css, cva, splitProps, memo)
  /node/           # Node.js APIs, config resolution, file watching
  /cli/            # CLI tool (@bamboocss/dev package)
  /parser/         # Static analysis and extraction
  /extractor/      # Expression evaluation behind the parser
  /ts-ast/         # AST backend on TypeScript 7's Go compiler, used by parser, extractor, node and vite
  /generator/      # Code generation for styled-system
  /config/         # Config loading and resolution
  /types/          # Type definitions (config options live here)
  /native-extractor/ # Rust/Oxc stylesheet extraction and static evaluation engine
  /vite/           # Vite plugin, including the build-time fold
  /plugin-*/       # vue and svelte are auto-injected; lightningcss is opt-in
  /preset-*/       # Design system presets (base, bamboo, atlaskit, open-props)
  /eslint-plugin/  # Lint rules
  /fixture/        # Shared test fixtures and utilities
  /logger/, /reporter/, /mcp/, /is-valid-prop/, /token-dictionary/

/sandbox/          # Integration tests and examples
  /codegen/        # Generated code validation tests (the scenario suites)
  /runtime-perf/   # Bundle-size and real-build assertions, on Vite 7 and Vite 8/Rolldown
 /vite-ts/, /astro/, /nuxt/, /svelte/, /solid-ts/,
 /preact-ts/, /qwik-ts/,
  /storybook/, /component-lib/    # per-framework integration apps
  /rsc/            # React Server Components through @vitejs/plugin-rsc; its test asserts that a
                   # sheet emitted by the server build is pruned against every environment

/website/          # Documentation site
```

Keep this list honest when packages come and go. A stale entry here is not cosmetic — it sends agents looking for files
that moved, and a removed suite documented as still running reads as coverage that does not exist.

### Key Concepts

1. **Static Extraction**: Bamboo analyzes source files to extract styles at build time
2. **Design Tokens**: Type-safe design tokens defined in config
3. **Recipes**: Reusable component style patterns (like variants)
4. **Conditions**: Responsive and state-based styling (e.g., `_hover`, `md:`, `_dark`)
5. **CSS Optimization**: Uses PostCSS (default) or LightningCSS (optional) for CSS processing

## Critical Rules

### 🚨 CSS Output is Sacred

**NEVER** accept changes that modify CSS output snapshots without explicit user approval:

- Run tests BEFORE and AFTER any dependency updates
- If snapshots change, investigate why and get user confirmation
- The test `packages/core/__tests__/atomic-rule.test.ts` is the primary CSS output validator
- CSS output consistency is more important than using latest package versions

### 🚨 Verify Before Reporting

**A pipeline's exit code is the last command's.** `pnpm check 2>&1 | tail -20` exits 0 whether the check passed or
failed, and `… | grep FAIL` exits 1 when it merely found nothing. Both were reported as results in one session — once as
a green check that was actually failing. Capture the real one:

```bash
pnpm check > /tmp/check.log 2>&1; echo "EXIT=$?"
```

**`tsc` colours its diagnostics, so `error` and `TS2345` are separated by escape codes.** Grepping a `pnpm typecheck`
run for `error TS` prints nothing whether or not it failed; one commit reached CI that way with a type error a plain
`pnpm typecheck; echo "EXIT=$?"` would have shown. Read the exit code, and the lines above it.

**`git checkout HEAD -- <file>` discards uncommitted work, with no reflog entry to recover it.** So does a `git stash`
whose flag is malformed — `--keep-index=false` is not a valid form, and the stash silently does not happen, which leaves
a "before" measurement that is really the "after" tree. Commit or copy to a scratch directory first, then confirm the
state actually changed (`grep` for something the change added) before trusting anything measured against it.

**With the TypeScript 7 backend, a file joining the shared project is a full program reload.** Membership is the
synthesized tsconfig's `files` list, so every `addSourceFile` of a path the project does not hold rewrites that list and
has the Go compiler re-derive a program the size of the whole inventory — about a second on 7,000 files — and it bumps
the tree revision every importer's cached import resolutions are checked against. The Vite compiler therefore must not
add a source per module it is handed: it skips modules outside `include`/`exclude` and framework rewrites that reach
nothing bamboo (`packages/vite/src/plugin.ts`, `compileModule`), and an auxiliary `X.__bamboo__.tsx` is a content event,
not a tree change (`packages/parser/src/project.ts`, `addSourceFile`). Contra's production build went from timing out at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gajus/bamboocss](https://github.com/gajus/bamboocss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
