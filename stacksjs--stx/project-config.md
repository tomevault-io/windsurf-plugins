---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**stx** is a fast, modern UI/templating framework that combines Laravel Blade-like syntax with Bun's performance. It's a monorepo containing multiple packages that work together to provide server-side rendering, component-based architecture, and a rich development experience.

## Core Architecture

### Monorepo Structure

This is a Bun workspace monorepo with packages in `packages/`:

- **`packages/stx`** - Core framework with template processing engine
- **`packages/bun-plugin`** - Bun plugin for `.stx` file processing
- **`packages/desktop`** - Native desktop application framework (NEW)
- **`packages/markdown`** - Markdown parsing with frontmatter support
- **`packages/sanitizer`** - HTML/XSS sanitization
- **`packages/iconify-core`** - Iconify integration core
- **`packages/iconify-generator`** - Icon package generation CLI
- **`packages/vscode`** - VS Code extension for `.stx` syntax
- **`packages/devtools`** - Development tooling
- **`packages/benchmarks`** - Performance benchmarks

**External dependency**: Craft (~/Code/craft) - Zig-based native webview framework for desktop/mobile apps

### Template Processing Pipeline

The core template processing is orchestrated by `packages/stx/src/process.ts` (~920 lines), which acts as a pipeline orchestrator delegating to extracted modules:

1. **Pre-processing**: Comments removal, escaped directives
2. **Directive Processing**: Sequential processing of directives in specific order:
   - Stack directives (`@push`, `@prepend`)
   - JavaScript/TypeScript execution (`@js`, `@ts`)
   - Includes and layouts (`@include`, `@layout`, `@extends`, `@section`)
   - Custom directives
   - Components (via `component-renderer.ts` and `component-registry.ts`)
   - Async components (`@async`)
   - Conditionals (`@if`, `@switch`, `@auth`, `@env`)
   - Loops (`@foreach`, `@for`)
   - Error boundaries (`@errorBoundary`, `@fallback`, `@enderrorBoundary`)
   - Memoization (`@memo`, `v-memo`)
   - Expressions (`{{ }}`, `{!! !!}`) — includes placeholder system for compile mode
   - i18n (`@translate`)
   - Forms (`@csrf`, `@method`, `@error`)
   - SEO directives (`@meta`, `@seo`)
3. **Post-processing**: Middleware, stack replacements, web component injection

#### Extracted Modules (from process.ts code split)

| Module | Responsibility |
|---|---|
| `signal-processing.ts` | Signal detection, setup function wrapping |
| `runtime-injection.ts` | Signals/router/browser runtime injection |
| `component-processing.ts` | Component tag parsing (findComponentTags, parseMultilineAttributes) |
| `script-validation.ts` | Client script validation rules |
| `inline-assets.ts` | `stx-inline` asset resolution |
| `misc-directives.ts` | `@json`, `@once`, ref attrs, `x-cloak` |

#### Signals Split

- `signals.ts` (~3075 lines) — runtime generation (template literal for client-side signals runtime)
- `signals-api.ts` (~550 lines) — TypeScript API (state, derived, effect, batch, lifecycle, type guards)

### Plugin System

The Bun plugin (`packages/bun-plugin/src/index.ts`) registers loaders for:
- `.stx` files - Processed as templates and exported as JavaScript modules
- `.md` files - Parsed with frontmatter and exported with `content` and `data` exports

## Development Commands

### Building

```bash
# Build all packages
bun run build

# Build individual packages
cd packages/bun-plugin && bun run build
cd packages/stx && bun run build
```

The build process:
1. Builds CSS assets (`packages/stx/scripts/build-css.ts`)
2. Compiles TypeScript using custom build scripts (`build.ts` in each package)
3. Creates compiled CLI binaries for multiple platforms

### Testing

```bash
# Run all tests
bun test

# Run tests for a specific package
cd packages/stx && bun test

# Run specific test file
bun test packages/stx/test/directives/conditionals.test.ts

# Run tests with coverage
cd packages/stx && bun test --coverage

# Run tests in watch mode
cd packages/stx && bun test --watch
```

Tests use Bun's built-in test runner with Happy DOM preloaded (configured in `bunfig.toml`). Test files follow the pattern `*.test.ts` and are located in each package's `test/` directory.

### Linting

```bash
# Lint all code
bun run lint

# Auto-fix linting issues
bun run lint:fix
```

Uses `@stacksjs/eslint-config` for consistent code style.

### Development Server

```bash
# Serve .stx files for development
bun packages/bun-plugin/dist/serve.js pages/ --port 8888

# Or using the CLI
stx-serve pages/ --port 3000
```

## Key Concepts

### Script Execution Model

**Only `<script server>` runs on the server.** All other script types run on the client:

| Tag | Execution | Purpose |
|-----|-----------|---------|
| `<script server>` | Server-side | Data fetching, variable extraction for templates |
| `<script>` | Client-side | Browser code, signals, composables |
| `<script client>` | Client-side | Same as bare `<script>` (explicit alias) |
| `<script type="module">` | Client-side | ES module scripts |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stacksjs/stx](https://github.com/stacksjs/stx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
