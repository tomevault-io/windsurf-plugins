---
trigger: always_on
description: Enables Claude to:
---

Here's a more concise and structured rewrite of your `CLAUDE.md`:

---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working with this repo.

---

## Project Overview

**NuxtPapier** is a minimal, SEO-friendly Nuxt 3 blog theme focused on content, performance, accessibility, and developer experience.

---

## Development

```bash
pnpm dev         # Start dev server
pnpm build       # Build for production
pnpm generate    # Generate static site
pnpm preview     # Preview production build
pnpm lint        # Run ESLint with auto-fix
pnpm typecheck   # TypeScript type checking
```

---

## Tools

### Playwright MCP
Enables Claude to:
* Navigate/interact with pages
* Capture screenshots
* Inspect/manipulate DOM
* Monitor network
* Generate tests
* Manage multiple tabs
* **IMPORTANT**: Never execute the Playwright MCP server yourself, only do it when specifically asked

### ast-grep
Use ast-grep for efficient code searching with structural patterns:

**Common patterns:**
```bash
# Find all ref declarations
ast-grep --pattern 'const $VAR = ref($$$)' --lang ts

# Find defineNuxtConfig
ast-grep --pattern 'defineNuxtConfig($_)' --lang ts

# Find computed properties
ast-grep --pattern 'const $VAR = computed(() => $$$)' --lang ts

# Find composables usage
ast-grep --pattern 'use$NAME($$$)' --lang ts

# Find component imports
ast-grep --pattern 'import { $$ } from "@/components/$$$"' --lang ts

# Find specific function calls
ast-grep --pattern '$OBJ.map($FUNC)' --lang ts

# Find async/await patterns
ast-grep --pattern 'await $PROMISE' --lang ts

# Find error handling patterns
ast-grep --pattern 'try { $$$ } catch' --lang ts

# Find route definitions
ast-grep --pattern 'definePageMeta({ $$$ })' --lang ts
```

**Supported languages:** ts, js, tsx, jsx, css, html, json, yaml

---

## Code Style

* **Imports**: Relative for locals, named preferred
* **Naming** (strictly enforced by ESLint `@typescript-eslint/naming-convention`):

  * **Variables**: 
    * camelCase by default
    * UPPER_SNAKE_CASE for exported/global constants
    * Boolean-named variables must start with `is`, `has`, `can`, `should`, `will`, `did`, `was`, `are`, `were`
      * Exceptions: `loading`, `enabled`, `disabled`, `open`, `closed`, `visible`, `hidden`, `active`, `menuOpen`
  * **Functions**: 
    * strictCamelCase enforced
    * Should start with verbs: `get`, `set`, `fetch`, `load`, `save`, `delete`, `update`, `create`, `handle`, `process`, `validate`, etc.
    * Event handlers must start with `on` or `handle`
    * Composables must start with `use`
  * **Types**: 
    * StrictPascalCase for all type-like constructs (classes, interfaces, types, enums)
    * Interfaces can optionally start with `I`
    * Type parameters: PascalCase with optional `T` prefix (single letters allowed)
    * Enum members: UPPER_CASE
  * **Parameters**: 
    * strictCamelCase required
    * Unused parameters must have underscore prefix `_`
  * **Properties**: 
    * strictCamelCase for object properties and methods
    * Private/protected class members must have underscore prefix `_`
    * Properties with special characters (CSS properties, @context, baseURL, etc.): no restrictions
  * **Imports**: camelCase or PascalCase allowed
* **Avoid**: `any`, `let`, `else`
* **Variables**: Use descriptive names (`searchQuery`, `userProfile`)
* **Comments**: Only for complex logic—favor self-explanatory code
* **Conditional Logic**: Always extract complex conditions to descriptive variables for readability
  * Avoid: `if (a == b && c === z)`
  * Prefer: `const isValidCondition = a == b && c === z; if (isValidCondition) { ... }`

---

## Architecture & Patterns

### Content

* Uses `@nuxt/content v3`
* Markdown files in `/content`
* Collections: `pages`, `posts`
* Reading time auto-calculated
* Drafts via `published: false`
* Config: `content.config.ts`

### Components

* Located in `/components/`, prefixed with `Base`
* `<script setup lang="ts">`, reactive prop destructuring
* Use TypeScript generics, no `withDefaults()`
* Order: script → template → style
* Styled with UnoCSS only

### Composables

* Prefixed with `use`
* Single-responsibility
* Expose error state
* Return readonly state when needed
* Fully typed

---

## VueUse Style Guide

* Import APIs from `vue`
* Prefer `ref`, use `shallowRef` for large objects
* Accept refs/computed/functions as args
* Use `tryOnUnmounted` for cleanup
* Return `isSupported`, `PromiseLike`, `controls` when relevant
* Allow config options (e.g. `flush`, `immediate`)

---

## Routing

* File-based routing
* Catch-all: `pages/[...path].vue`
* Homepage: `pages/(home).vue`
* Blog posts: `/posts/[postSlug]`
* Custom 404 supported

---

## Styling

* UnoCSS + Wind4 preset
* Typography plugin for prose
* Theming via CSS variables (light/dark)
* Config: `uno.config.ts`

---

## Server Features

* RSS: `/server/routes/rss.xml.ts`
* Sitemap: `/server/routes/sitemap.xml.ts`
* Robots: `/server/routes/robots.txt.ts`
* APIs: `/server/api/`

---

## Key Files

* `app.config.ts`: Site metadata
* `nuxt.config.ts`: Nuxt modules/config
* `uno.config.ts`: CSS config
* `content.config.ts`: Content parsing rules
* `constants.ts`: Constants (social links, etc.)
* `types/index.d.ts`: Shared types

---

## Error Handling

Use standard JavaScript error handling patterns:

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexanderop/NuxtPapier](https://github.com/alexanderop/NuxtPapier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
