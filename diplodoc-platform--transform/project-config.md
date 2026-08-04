---
trigger: always_on
description: This file contains instructions for AI agents working with the `@diplodoc/transform` project.
---

# AGENTS.md

This file contains instructions for AI agents working with the `@diplodoc/transform` project.

## Common Rules and Standards

**Important**: This package follows common rules and standards defined in the Diplodoc metapackage. When working in metapackage mode, refer to:

- **`.agents/style-and-testing.md`** in the metapackage root for:
  - Code style guidelines
  - **Language requirements** (commit messages, comments, docs MUST be in English)
  - Commit message format (Conventional Commits)
  - Pre-commit hooks rules (**CRITICAL**: Never commit with `--no-verify`)
  - Testing standards
  - Documentation requirements
- **`.agents/core.md`** for core concepts
- **`.agents/monorepo.md`** for workspace and dependency management
- **`.agents/dev-infrastructure.md`** for build and CI/CD

**Note**: In standalone mode (when this package is used independently), these rules still apply. If you need to reference the full documentation, check the [Diplodoc metapackage repository](https://github.com/diplodoc-platform/diplodoc).

## Project Description

`@diplodoc/transform` is the core transformation package for the Diplodoc platform. It converts Yandex Flavored Markdown (YFM) to HTML, providing both server-side transformation and client-side runtime components.

**Key Features**:

- **YFM to HTML transformation** — Core markdown processing with YFM extensions
- **Plugin system** — Extensible architecture using extension packages
- **Client-side runtime** — Interactive components (tabs, cuts, terms, etc.)
- **SCSS styles** — Comprehensive styling system with CSS variables
- **Multiple output formats** — HTML, metadata, titles, headings
- **HTML sanitization** — Built-in XSS protection with customizable sanitizer

> **Note**: Liquid template support and YFMLint integration are deprecated in this package. Use separate packages (`@diplodoc/liquid` and `@diplodoc/yfmlint`) instead. The legacy code still exists in `src/transform/liquid/` and `src/transform/yfmlint/` for backward compatibility but should not be used in new projects.

**Primary Use Case**: Core transformation engine for the Diplodoc documentation platform. Used by `@diplodoc/cli` and other tools to process YFM documents into HTML with interactive features.

## Project Structure

### Main Directories

- `src/` — source code
  - `transform/` — **Server-side transformation** (Node.js)
    - Core transformation logic and MarkdownIt initialization
    - `plugins/` — Built-in plugins and extension integrations (cut, tabs, file, anchors, code, notes, term, table, etc.)
    - `liquid/` — Liquid template engine implementation (deprecated, use `@diplodoc/liquid`)
    - `preprocessors/` — Content preprocessing
    - `yfmlint/` — Custom markdownlint rules (deprecated, use `@diplodoc/yfmlint`)
    - HTML sanitization and utility functions
  - `js/` — **Client-side runtime** (browser)
    - Interactive components for anchor handling, code blocks, inline code, terms, wide mode, print functionality
    - Runtime code for browser execution
  - `scss/` — **Styles** (SCSS)
    - Main stylesheets (forwards extension styles)
    - Base styles, component styles, and print-specific styles
- `test/` — **Unit tests** (Vitest)
  - Test files for transform functionality, plugins, and HTML output snapshots
  - Legacy tests for Liquid and YFMLint (deprecated)
- `e2e/` — **E2E tests** (Playwright, separate package)
  - Visual regression and integration tests
  - Storybook-based test fixtures
- `playground/` — **Interactive playground** (separate package)
  - Published to gh-pages for interactive YFM syntax testing
- `lib/` — compiled server-side code (generated, CommonJS)
- `dist/` — compiled client-side assets (generated)
  - Bundled JavaScript, compiled CSS, and source SCSS files

### Configuration Files

- `package.json` — package metadata and dependencies
- `tsconfig.json` — TypeScript configuration (development, includes test/)
- `tsconfig.transform.json` — TypeScript configuration (for building lib/)
- `vitest.config.mjs` — Vitest configuration
- `CHANGELOG.md` — change log (managed by release-please)
- `CONTRIBUTING.md` — contribution guidelines

## Architecture: Server vs Client

This package has a **dual architecture**:

### Server-Side (`src/transform/`)

- **Purpose**: Transform YFM to HTML on the server
- **Runtime**: Node.js
- **Output**: `lib/` directory (CommonJS)
- **Entry**: `lib/index.js`
- **Dependencies**: Uses extension packages as plugins
  - `@diplodoc/cut-extension` — cut plugin
  - `@diplodoc/tabs-extension` — tabs plugin
  - `@diplodoc/file-extension` — file plugin

**How extensions are used**:

- Extensions provide `transform()` function that returns a MarkdownIt plugin
- Transform imports and uses these plugins in `src/transform/plugins.ts`
- Example: `src/transform/plugins/cut.ts` imports from `@diplodoc/cut-extension`

### Client-Side (`src/js/` and `src/scss/`)

- **Purpose**: Interactive behavior and styling in the browser
- **Runtime**: Browser
- **Output**: `dist/` directory (bundled JS and CSS)
- **Dependencies**: Uses extension runtime components
  - `@diplodoc/cut-extension/runtime` — cut runtime
  - `@diplodoc/tabs-extension/runtime` — tabs runtime

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diplodoc-platform/transform](https://github.com/diplodoc-platform/transform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
