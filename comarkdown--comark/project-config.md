---
trigger: always_on
description: This document provides guidance for AI agents working on the comark monorepo.
---

# Agent Instructions

This document provides guidance for AI agents working on the comark monorepo.

## Project Overview

This is a **monorepo** containing multiple packages related to Comark (Components in Markdown) syntax parsing. The main package is `comark`.

**comark** is a Components in Markdown (Comark) parser that extends standard Markdown with component syntax. It provides:

- Fast synchronous and async parsing via markdown-it
- Streaming support for real-time/incremental parsing
- Vue, React and Svelte renderers
- Syntax highlighting via Shiki
- Auto-close utilities for incomplete markdown (useful for AI streaming)

## Monorepo Structure

```
/                         # Root workspace
├── packages/             # All publishable packages
│   ├── comark/           # Main Comark parser + core plugins
│   ├── comark-html/      # HTML renderer (@comark/html)
│   ├── comark-ansi/      # ANSI terminal renderer (@comark/ansi)
│   ├── comark-vue/       # Vue renderer + plugins (@comark/vue)
│   ├── comark-react/     # React renderer + plugins (@comark/react)
│   ├── comark-svelte/    # Svelte renderer + plugins (@comark/svelte)
│   └── comark-nuxt/      # Nuxt module (@comark/nuxt)
├── examples/             # Example applications
│   ├── 1.vue-vite/       # Vue + Vite + Tailwind CSS v4
│   ├── 2.react-vite/     # React 19 + Vite + Tailwind CSS v4
│   └── 3.plugins/        # Plugin examples (vue-vite-math, vue-vite-mermaid)
├── docs/                 # Documentation site (Docus-based)
├── scripts/              # Build/sync scripts
├── pnpm-workspace.yaml   # Workspace configuration
├── tsconfig.json         # Root TypeScript config
├── eslint.config.mjs     # ESLint configuration
└── package.json          # Root package (private, scripts only)
```

## Package: comark

Located at `packages/comark/`:

```
packages/comark/
├── src/
│   ├── index.ts              # Core parser: parse(), autoCloseMarkdown()
│   ├── render.ts             # String rendering: renderMarkdown() (renderHTML moved to @comark/html)
│   ├── types.ts              # TypeScript interfaces (ParseOptions, etc.)
│   ├── ast/                  # Comark AST types and utilities
│   │   ├── index.ts          # Re-exports (comark/ast entry point)
│   │   ├── types.ts          # ComarkTree, ComarkNode, ComarkElement, ComarkText
│   │   └── utils.ts          # textContent(), visit() tree utilities
│   ├── plugins/              # Built-in and optional plugins
│   │   ├── alert.ts          # Alert/callout blocks
│   │   ├── emoji.ts          # Emoji shortcodes
│   │   ├── highlight.ts      # Syntax highlighting via Shiki (peer: shiki)
│   │   ├── math.ts           # LaTeX math via KaTeX (peer: katex)
│   │   ├── mermaid.ts        # Mermaid diagrams (peer: beautiful-mermaid)
│   │   ├── security.ts       # XSS/security sanitization
│   │   ├── summary.ts        # Summary extraction
│   │   ├── task-list.ts      # GFM task lists
│   │   └── toc.ts            # Table of contents
│   └── internal/             # Internal implementation (not exported)
│       ├── front-matter.ts
│       ├── parse/            # Parsing pipeline
│       └── stringify/        # AST → string rendering
├── test/                 # Vitest test files
├── package.json
└── tsconfig.build.json
```

### Peer dependencies

| Peer | Required by |
|------|-------------|
| `shiki` | `comark/plugins/highlight` |
| `katex` | `comark/plugins/math` |
| `beautiful-mermaid` | `comark/plugins/mermaid` |

All are optional — only install what you use.

## Package: @comark/html

Located at `packages/comark-html/`. Framework-free HTML string rendering.

### Exports

```json
{
  ".": "./dist/index.js",
  "./plugins/*": "./dist/plugins/*.js",
  "./render": "./dist/render.js"
}
```

### Usage

```typescript
import { render, renderHTML, createRender } from '@comark/html'
import highlight from '@comark/html/plugins/highlight'
import math, { Math } from '@comark/html/plugins/math'

// Flat options — ParseOptions & RenderOptions merged at top level
const renderFn = createRender({
  plugins: [highlight({ themes: { light: 'github-light', dark: 'github-dark' } })],
  components: {
    Math,
    alert: ([, attrs, ...children], { render }) =>
      `<div class="alert alert-${attrs.type}">${render(children)}</div>`
  },
})

const html = await renderFn(markdownString)
```

---

## Package: @comark/ansi

Located at `packages/comark-ansi/`. ANSI terminal renderer.

### Exports

```json
{
  ".": "./dist/index.js",
  "./plugins/*": "./dist/plugins/*.js",
  "./render": "./dist/render.js"
}
```

### Usage

```typescript
import { log, render, renderANSI, createLog, createRender } from '@comark/ansi'
import highlight from '@comark/ansi/plugins/highlight'
import math, { Math } from '@comark/ansi/plugins/math'

// Flat options — ParseOptions & RenderANSIOptions merged at top level
const logFn = createLog({
  plugins: [highlight(), math()],
  components: { Math },
  width: 120,                      // terminal width
  colors: true,                    // emit ANSI escape codes
  write: (s) => process.stderr.write(s),
})

await logFn(markdownString)
```

---

## Package: @comark/vue

Located at `packages/comark-vue/`. Vue 3 renderer with framework-specific plugin wrappers.

```
packages/comark-vue/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [comarkdown/comark](https://github.com/comarkdown/comark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
