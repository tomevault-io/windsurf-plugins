---
trigger: always_on
description: 1. **Public Repository Warning**: This is a PUBLIC repository. Never include internal/sensitive information in the project.
---

# Agent Rules

1. **Public Repository Warning**: This is a PUBLIC repository. Never include internal/sensitive information in the project.

2. **File Operations**:

   - Always read files before editing them
   - Always prefer editing existing files instead of creating new ones
   - Always use specialized tools (read_files, edit, write) instead of bash commands where possible
   - Never create documentation files unless directed

3. **Communication Style**:

   - Always be concise and technical
   - Always output text directly to user, NOT via bash echo or comments
   - Always request more information when necessary
   - Never use emoji

4. **Code Quality**:

   - Always run relevant tests for the area of code being modified
   - Always improve the test suite when fixing issues instead of just reading the code or using CLI
   - Always format files after editing using the relevant tool
   - Never use placeholders in code, always use real values or ask for them

5. **Git Safety**:

   - Never push or commit unless directed
   - Never update git config
   - Never force push to main/master
   - Never use interactive git commands (`-i` flag)
   - Never skip hooks (--no-verify, --no-gpg-sign)
   - Never run destructive git commands unless directed
   - Never create PRs unless directed

6. **Task Management**:

   - Always use the todo/task tool
   - Always mark todos/tasks as complete immediately after finishing them
   - Never work on multiple todos/tasks simultaneously

7. **When stuck**:
   - Check existing documentation
   - Ask the developer for clarification
   - Review recent commits for context

# Project Overview

Compiled is a **compile-time CSS-in-JS library** for React that transforms styled components into atomic CSS at build time via Babel AST transformations. Unlike runtime CSS-in-JS libraries, all styles are extracted and processed during compilation.

**Core principle**: Zero runtime CSS generation. All styles are determined at build time through static analysis and AST manipulation.

# Architecture

## Monorepo Structure

Yarn workspaces monorepo with three main workspace types:

- `packages/*` - Core libraries published to npm
- `examples/*` - Example integrations (webpack, parcel, ssr)
- `fixtures/*` - Test fixtures and babel component examples

**Critical packages**:

- `packages/babel-plugin/` - Core Babel transformation engine (AST processing)
- `packages/react/` - Runtime library with `ax`, `ix`, `CC`, `CS` utilities
- `packages/css/` - CSS parsing, sorting, and atomic stylesheet generation
- `packages/webpack-loader/`, `packages/parcel-transformer/` - Bundler integrations
- `packages/babel-plugin-strip-runtime/` - Extracts styles to separate `.compiled.css` files
- `packages/codemods/` - Migration tools (emotion→compiled, styled-components→compiled)
- `packages/eslint-plugin/` - ESLint rules for enforcing Compiled best practices
- `packages/jest/` - Jest testing utilities and matchers
- `packages/utils/` - Shared utilities across packages
- `packages/benchmark/` - Performance benchmarking tools

## Build System

Uses TypeScript with **ttsc** (TypeScript Transformer Compiler) instead of plain `tsc`:

```bash
# Build order matters - babel fixtures first, then browser/cjs/esm
yarn build:babel-fixture  # Compile test fixtures
yarn build:cjs            # CommonJS output (packages/tsconfig.json)
yarn build:esm            # ES modules (packages/tsconfig.esm.json)
yarn build:browser        # Browser bundles (packages/tsconfig.browser.json)
```

Post-build: `scripts/postbuild.sh` removes `__tests__/` and `__perf__/` from dist.

## Atomic CSS System

**Key concept**: One CSS rule per declaration. Class names encode both the atomic group (property+selectors+media) and value.

Format: `_{group}{value}` where group is 4 chars and value is 4+ chars (e.g., `_1wyb1fwx` for `font-size:12px`). The underscore + 4-char group makes `ATOMIC_GROUP_LENGTH = 5` in code.

**Runtime utilities** (`packages/react/src/runtime/`):

- `ax()` - Merges classnames, ensures last atomic group wins (handles composition)
- `ac()` - Advanced merging with AtomicGroups (chainable, memoized)
- `ix()` - Injects CSS variables for dynamic values
- `CC`, `CS` - Components for style sheet injection

**Auto-ordering**: Pseudo-classes sorted to prevent cascade issues (`:link` → `:visited` → `:focus-within` → `:focus` → `:hover` → `:active`)

# User-Facing APIs

Main exports from `@compiled/react`:

- `styled` - Create styled components (e.g., `styled.div({ color: 'red' })`)
- `css` - Define styles for css prop (e.g., `css({ color: 'red' })`)
- `cssMap` - Create CSS maps for conditional styling
- `ClassNames` - Render prop component for dynamic className generation
- `keyframes` - Define CSS animations
- `xcss` / `cx()` - Strict-typed CSS system (Atlassian design system integration)
- `createStrictAPI()` - Create type-safe CSS APIs with restricted properties

See `packages/react/src/index.ts` for all exports.

# Critical Workflows

## Development

```bash
yarn install              # Install + postinstall.sh (sets up tooling)
yarn start                # Storybook on :6006 (watch babel-plugin changes)
yarn test <filter> --watch  # Run specific tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atlassian-labs/compiled](https://github.com/atlassian-labs/compiled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
