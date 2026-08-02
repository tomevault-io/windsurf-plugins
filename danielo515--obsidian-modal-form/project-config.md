---
trigger: always_on
description: Obsidian Modal Form is a plugin for [Obsidian](https://obsidian.md) that lets users define forms for structured data entry, openable from anywhere JavaScript can run (Templater, QuickAdd, DataviewJS, etc.). Forms are defined as JSON-like structures, rendered as modals with Svelte components, and return typed results.
---

# CLAUDE.md - Obsidian Modal Form

## Project Overview

Obsidian Modal Form is a plugin for [Obsidian](https://obsidian.md) that lets users define forms for structured data entry, openable from anywhere JavaScript can run (Templater, QuickAdd, DataviewJS, etc.). Forms are defined as JSON-like structures, rendered as modals with Svelte components, and return typed results.

**Current version:** 1.61.1
**License:** MIT
**Min Obsidian version:** 0.15.0

## Build & Development Commands

```bash
npm run dev        # Watch mode — outputs to EXAMPLE_VAULT/.obsidian/plugins/modal-form/
npm run build      # Full build: lint + svelte-check + production bundle → main.js
npm run check      # Lint (ESLint) + svelte-check (no build)
npm run lint       # ESLint only
npm run test       # Jest (all tests in src/)
npm run test-w     # Jest in watch mode
```

**CI runs `npm run build && npm run test` on every pull request.** Both must pass.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | TypeScript 5.5 (strict mode, ES2019 target) |
| UI framework | Svelte 4 |
| Bundler | esbuild (CJS output) |
| Testing | Jest + ts-jest |
| Linting | ESLint + @typescript-eslint + fp-ts plugin + @stylistic |
| Formatting | Prettier (100 char width, trailing commas) |
| Validation | Valibot (schema-based) |
| FP library | fp-ts (Option, Either, TaskEither, pipe/flow) |
| Template parsing | parser-ts (parser combinators) |

## Repository Structure

```
src/
├── main.ts                    # Plugin entry point (extends Obsidian Plugin)
├── API.ts                     # Public API exposed globally as modalForms
├── FormModal.ts / .svelte     # Modal wrapper + Svelte form UI
├── ModalFormSettingTab.ts      # Plugin settings tab
├── core/                      # Business logic (UI-independent)
│   ├── formDefinition.ts      # Core types: FormDefinition, FieldDefinition, etc.
│   ├── formDefinitionSchema.ts # Valibot schemas + migration logic
│   ├── FormBuilder.ts         # Fluent API for programmatic form creation
│   ├── FormResult.ts          # Result wrapper (frontmatter, dataview, template output)
│   ├── settings.ts            # Plugin settings types/defaults
│   ├── input/                 # Input type schemas and field logic
│   │   ├── InputDefinitionSchema.ts  # Valibot schemas for all input types
│   │   └── dependentFields.ts        # Conditional field visibility
│   ├── template/              # Template system
│   │   ├── templateParser.ts  # Parser-ts template parser ({{ var }}, {# commands #})
│   │   ├── TemplateService.ts # Template service interface
│   │   ├── TemplaterService.ts # Templater plugin integration
│   │   └── BasicTemplateService.ts
│   └── files/                 # File service abstraction
├── store/                     # State management
│   ├── formEngine.ts          # Svelte store-based form state machine
│   └── SettngsStore.ts        # Global settings store
├── views/                     # Obsidian views + Svelte components
│   ├── components/
│   │   ├── Form/              # Field-level input components (one per type)
│   │   └── FormBuilder/       # UI for creating/editing form definitions
│   ├── EditFormView.ts        # View for editing a form definition
│   ├── ManageFormsView.ts     # View for listing/managing forms
│   └── FormImportView.ts      # Form import/export
├── std/                       # FP utilities (re-exports fp-ts + custom helpers)
│   ├── index.ts               # Main exports: pipe, flow, O, E, A, TE, parse, etc.
│   ├── Array.ts               # Extended array utilities
│   └── TaskEither.ts          # Extended TE utilities
├── suggesters/                # Obsidian modal pickers
├── utils/                     # Logging, errors, file helpers
└── typings/                   # Ambient type declarations
```

Key non-src files:
- `esbuild.config.mjs` — Build configuration
- `manifest.json` / `versions.json` — Obsidian plugin metadata
- `styles.css` — Plugin styles
- `EXAMPLE_VAULT/` — Dev vault for testing (dev mode output target)

## Code Conventions

### TypeScript

- **Strict mode** is on: `noImplicitAny`, `strictNullChecks`, `noUncheckedIndexedAccess`.
- **Path aliases:** Use `@std` for `src/std` and `@core` for `src/core`.
- **Indent:** 4 spaces (enforced by `.editorconfig`).
- **Print width:** 100 characters (Prettier).
- **Arrow parens:** Always required — `(x) => x`, not `x => x`.
- **Trailing commas:** Always (`"all"`).

### Functional Programming (fp-ts)

This codebase uses fp-ts extensively. Key conventions:

- **Namespace imports** — Import fp-ts modules as single-letter namespaces:
  - `O` = `fp-ts/Option`
  - `E` = `fp-ts/Either`
  - `A` = `fp-ts/Array`
  - `TE` = `fp-ts/TaskEither`
  - `RTE` = `fp-ts/ReaderTaskEither`
  - `S` = `fp-ts/string`
  - `pipe` and `flow` from `@std`
- **No direct library imports** — ESLint rule `fp-ts/no-lib-imports` is enforced. Always use namespace imports (e.g., `O.some(x)` not `import { some } from "fp-ts/Option"`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielo515/obsidian-modal-form](https://github.com/danielo515/obsidian-modal-form) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
