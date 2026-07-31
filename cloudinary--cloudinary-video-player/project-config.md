---
trigger: always_on
description: This document provides guidance for AI agents working on this codebase. Follow these guidelines for all code changes.
---

# Agent Notes for Cloudinary Video Player

This document provides guidance for AI agents working on this codebase. Follow these guidelines for all code changes.

## Your Role

You are an expert software engineer working on the Cloudinary Video Player - a Video.js-based player with Cloudinary integration, plugins, and multi-format output (UMD + ESM). You write clean, maintainable code that follows existing patterns. You cannot see visual results, so you rely on user testing and feedback.

## Project Knowledge

**Tech Stack:** Video.js 8.x, webpack 5, Vitest (unit), Playwright (e2e), ESLint, Babel. Source is JavaScript (no TypeScript in src).

**File Structure:**
- `src/` - Application source (main entry: `src/index.js`, player: `src/video-player.js`)
- `src/plugins/` - Video.js plugins (cloudinary, playlist, chapters, adaptive-streaming, etc.)
- `src/components/` - UI components
- `src/utils/` - Shared utilities
- `dist/` - UMD bundles (CDN, legacy) - do not edit directly
- `lib/` - ESM output (tree-shakeable) - do not edit directly
- `test/unit/` - Vitest unit tests
- `test/e2e/` - Playwright e2e tests
- `docs/` - Demo pages and documentation

## Commands

Use these to validate changes. Prefer file-scoped commands when possible to avoid slow full builds.

```bash
# Lint source
npm run lint

# Unit tests - full suite
npm run test:unit

# Unit tests - single file
npm run vitest run --config .config/vitest.config.ts test/unit/path/to/file.test.js

# E2E tests (use sparingly)
npm run test:e2e

# Build (use when explicitly needed)
npm run build
npm run build-es
```

## Boundaries

- **Always do:** Follow existing patterns in the file, minimize diffs, run eslint on changed files, suggest tests for new features
- **Ask first:** Adding dependencies, modifying webpack config, changing package.json exports, major refactors
- **Never do:** Edit `dist/` or `lib/` (generated), fix unrelated lint errors unless requested, commit secrets, refactor unrelated code "while you're there"

---

## Key Principles & Coding Style

### Core Philosophy
- **KISS (Keep It Simple)**: Prefer simple, straightforward solutions over clever ones
  - If you're tempted to write "clever" code, step back and find the simpler approach
  - Simple code is easier to read, maintain, and debug
  - Avoid unnecessary abstractions or premature optimizations
- **Minimize diffs**: Make the smallest change that solves the problem
  - Only touch code directly related to the task
  - Don't refactor unrelated code "while you're there"
  - Smaller diffs are easier to review, test, and debug
- **Single responsibility**: Each function does one thing well
  - Don't combine multiple concerns in one function
  - Clear function names that describe exactly what they do
- **DRY (Don't Repeat Yourself)**: Extract duplicated logic into shared helpers
  - When the same logic appears in multiple places, centralize it in `src/utils/` or a shared module
  - Prefer one source of truth over copy-paste
- **Elegant code**: Clean, readable, maintainable
  - Code should read like well-written prose
  - Future maintainers (including you) will thank you

### Code Quality Guidelines
- **Avoid redundant checks**: Don't use `||` fallbacks if options are already normalized
  - Bad: `const value = options.foo || options.bar || defaults.foo;`
  - Good: If options are normalized, just use `options.foo`
  - If unsure, check where normalization happens rather than adding defensive code
- **Flatten logic**: Avoid nested conditionals when possible
  - Use early returns for error conditions
  - Extract complex conditions into well-named boolean variables
  - Prefer single exit points when reasonable (but don't force it)
- **Don't fix unrelated lint errors** unless explicitly requested
  - Unrelated fixes make diffs harder to review
  - If you discover a bug, add a TODO comment instead of fixing it
- **Suggest tests**: Recommend adding tests where missing or appropriate for new features or significant changes. After completing implementation, proactively ask: "Would you like me to add tests for this?" Before proposing new tests, search for existing coverage in `test/unit/` and `test/e2e/` to avoid duplication.

### Working with Existing Code
- **Respect existing patterns**: Follow the style and patterns already in the file
  - Match indentation, naming conventions, and code structure
- **Understand before changing**: Read surrounding code to understand context
  - Why was it written this way?
  - What might break if you change it?
  - Are there tests that cover this code?
- **Don't over-engineer**: The simplest solution is usually the best
  - Resist the urge to build flexible, generic solutions for specific problems
  - YAGNI (You Aren't Gonna Need It) - don't add features "just in case"
- **Challenge duplication**: When new functionality spans multiple plugins or components, centralize logic in `src/utils/` or shared helpers instead of scattering copies.
- **Value deletion**: Actively seek opportunities to remove code. A problem solved by deleting code is superior to one solved by adding it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudinary/cloudinary-video-player](https://github.com/cloudinary/cloudinary-video-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
