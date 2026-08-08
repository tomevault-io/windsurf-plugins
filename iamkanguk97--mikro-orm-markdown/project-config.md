---
trigger: always_on
description: Generates Mermaid ERD diagrams and Markdown documentation from MikroORM entity metadata.
---

# mikro-orm-markdown — Agent Guide

Generates Mermaid ERD diagrams and Markdown documentation from MikroORM entity metadata.
Supports a programmatic API (`generateMarkdown`) and a CLI (`mikro-orm-markdown`).

## Tech Stack

| Layer | Tool |
|---|---|
| Language | TypeScript (ESM, `"type": "module"`) |
| Linter / Formatter | Biome (`biome.jsonc`) |
| Type checker | `tsc` via `tsconfig.json` / `tsconfig.test.json` |
| Test runner | Vitest |
| Bundler | tsup (CJS + ESM dual output → `dist/`) |
| Commit hook | Husky + commitlint |
| AST / JSDoc parsing | ts-morph |

## Source Layout

```text
src/
  cli.ts            # Commander-based CLI entry point
  index.ts          # Public API: generateMarkdown(), resolveJsDocSources()
  defaults.ts       # Defaults shared by the CLI and the programmatic API
  error-chain.ts    # causeChain() / errorMessage() helpers for unknown errors
  messages.ts       # Structured warning/error payloads (StructuredMessage, StructuredError)
  provider.ts       # Auto-injects TsMorphMetadataProvider when needed
  source-path.ts    # Canonicalizes entity source paths (symlinks, parent segments)
  docs/
    jsdoc.ts        # Parses JSDoc tags from .ts entity files via ts-morph
  metadata/
    load.ts         # Initialises MikroORM and extracts EntityMetadata[]
    renderable.ts   # Shared predicate: which entities appear in the document
    extends.ts      # Normalizes meta.extends across MikroORM majors (v6 name / v7 class)
  model/
    types.ts        # Internal model types (EntityModel, ColumnModel, RelationEdge, …)
    diagram.ts      # Converts EntityMetadata[] → DiagramModel (boxes, columns, edges, constraints)
    build.ts        # Merges DiagramModel + JsDocResult → DocumentModel (namespace groups)
  render/
    markdown.ts     # Renders DocumentModel → Markdown string
    mermaid.ts      # Renders DiagramModel → erDiagram fences (+ optional frontmatter)
    column-markers.ts # Column-marker labels shared by both renderers
    escape.ts       # Mermaid / Markdown string escaping helpers
```

## Development Commands

```bash
npm run build          # Production build → dist/
npm run dev            # Watch mode build
npm run lint           # Biome check (no auto-fix)
npm run lint:fix       # Biome check with auto-fix
npm run format:check   # Biome format check only
npm run format         # Biome format with auto-fix
npm run typecheck      # tsc --noEmit
npm run test           # Vitest run (all tests)
npm run test:watch     # Vitest watch
npm run test:coverage  # Vitest + v8 coverage
npm run test:pack      # Smoke-test the npm tarball + CLI binary
npm run example:erd    # Build then generate ERD.md from examples/entities/
```

**Full pre-release check sequence:**

```bash
npm run lint && npm run format:check && npm run typecheck && npm run test && npm run build && npm run test:pack
```

## Test Layout

```text
test/
  cli.test.ts              # CLI option parsing and validation
  error-chain.test.ts      # causeChain()/errorMessage() unit tests
  messages.test.ts         # StructuredMessage / emitWarning unit tests
  provider.test.ts         # TsMorphMetadataProvider auto-injection unit tests
  source-path.test.ts      # Source path canonicalization unit tests
  e2e/cli-smoke.test.ts    # End-to-end: spawns the built CLI binary
  integration/generate.test.ts  # generateMarkdown() integration tests
  docs/jsdoc.test.ts       # JSDoc parsing unit tests
  metadata/
    load.test.ts           # Metadata loading unit tests
    renderable.test.ts     # Renderable-entity predicate unit tests
    extends.test.ts        # meta.extends normalization unit tests
  model/
    diagram.test.ts        # Diagram model builder unit tests
    build.test.ts          # Document model builder unit tests
  render/
    markdown.test.ts       # Markdown renderer unit tests
    mermaid.test.ts        # Mermaid renderer unit tests
    mermaid-parser.ts      # Shared helper: parses fences with the official Mermaid parser
    mermaid-parser.test.ts # Contract tests for generated fences against that parser
  helpers/                 # Shared test helpers (temp dirs, fixture pipeline, factories, paths)
  fixtures/                # Fixture entities and MikroORM configs
```

When adding a feature, add tests to the matching file. New rendering behaviour belongs in `render/*.test.ts`; new diagram-model behaviour in `model/diagram.test.ts`; new document-model behaviour in `model/build.test.ts`. Reuse the shared helpers in `test/helpers/` instead of re-declaring temp dirs, fixture pipelines, or metadata factories.

## Generation Pipeline

```text
MikroORM config
  └─ loadEntityMetadata()       → EntityMetadata[]  (metadata/load.ts)
  └─ loadJsDoc()                → JsDocResult        (docs/jsdoc.ts)
  └─ buildDocumentModel()       → DocumentModel      (model/build.ts)
         └─ buildDiagramModel() → DiagramModel       (model/diagram.ts)
  └─ renderMarkdown()           → string             (render/markdown.ts)
         └─ renderErDiagram()   → erDiagram fence    (render/mermaid.ts)
```

## Supported JSDoc Tags

| Tag | Scope | Effect |
|---|---|---|
| `@namespace <name>` | Entity class | Groups entity in both ERD and text table |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamkanguk97/mikro-orm-markdown](https://github.com/iamkanguk97/mikro-orm-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
