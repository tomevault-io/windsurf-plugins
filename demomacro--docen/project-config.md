---
trigger: always_on
description: > Coding standards, design patterns, and the contribution workflow live in [CONTRIBUTING.md](./CONTRIBUTING.md). This file is the architectural context an agent must understand before changing code. Read both.
---

> Coding standards, design patterns, and the contribution workflow live in [CONTRIBUTING.md](./CONTRIBUTING.md). This file is the architectural context an agent must understand before changing code. Read both.

## Project

**docen** is a monorepo for online Office editors.

- **`docen`** — all-in-one aggregate entry: re-exports `@docen/docx` (converters/engine, via `docen/docx`) and `@docen/editor` (`<docen-document>` via `docen/editor`). One dependency covers both headless conversion and the full editor; the root entry stays side-effect-free so converter-only imports remain tree-shakable.
- **`@docen/vue`** — Vue 3 adapter for `@docen/editor`: a typed `<DocenDocument>` component (`v-model` content + `v-slot="{ editor }"` + template-ref expose). `vue` is a peer dependency and `@docen/editor` a regular dependency, so the Vue surface stays isolated from the framework-neutral core.
- **`@docen/editor`** — multi-editor assembly: a Fluent UI host (`<docen-workspace>` + UI surfaces) shared by the editor elements `<docen-document>` and `<docen-presentation>` (today) plus the `<docen-workbook>` stub (future); all UI surfaces (title-bar/ribbon/status-bar/panes) and engine extensions are contributed by **add-ins** (Office.js-style). Bundles the `@docen/docx` engine; owns the canvas stage, painting, and caret/selection mapping.
- **`@docen/docx`** — the engine: Tiptap DOCX schema + converters + custom extensions + the layout projection (Tiptap JSON → LayoutDoc, incl. WMF/EMF+ metafile replay). No UI.
- **`@docen/pptx`** — the PPTX engine: re-exports the OOXML parse/generate surface from `@office-open/pptx` and projects `PresentationOptions` into the drawing members the core painter paints (`scene/` → `projectPresentation`). No UI.
- **`@docen/markdown`** — the format-agnostic Markdown syntax layer: parses Markdown into a neutral IR (heading/nested lists/tables/quotes/inline marks) and renders it back. Format packages implement `MarkdownMapper<T>` to bind their own model; `@docen/docx` ships the reference mapper and re-exports the one-argument `parseMarkdown`/`generateMarkdown`.
- **`@docen/layout`** — the layout engine: block/flow/text measurement and pagination producing a paginated `LayoutDoc`. Pure computation, no DOM, no editor types.
- **`@docen/pretext`** — vendored fork of `@chenglou/pretext` 0.0.8 (text measurement & line breaking), maintained in-tree because docen's Word/CJK `edit == render` fixes (CJK canvas→DOM advance correction, empty-text atom retention) are deeper than a patch file carries. Consumed by `@docen/layout` (line breaking) and `@docen/editor` (paginator measurement).
- **`@docen/core`** — the scene painter package: LayoutDoc → LeaferJS tree, consumed by the editors' canvas stages. No layout decisions, no editing semantics.
- **`leafer-x-metafile`** — zero-dependency WMF/EMF+ metafile replay into neutral drawing members (no Leafer, no docen types — built to be contributed to the LeaferJS ecosystem as `leafer-x-*`). The docx layout projection consumes it and adapts members into `LayoutDoc`.
- **`@docen/deduplicate`** — document comparison (SimHash + Winnowing fingerprinting, `compareDocuments`/`findDuplicates`) for the editors' future compare feature. Standalone; no editor dependencies.
- **`@office-open/*`** — OOXML parse/generate APIs (external). The canonical document model.

The `xlsx` workbook editor is the last unimplemented one (`packages/editor/src/workbook.ts` is a stub). It will reuse the same host + add-in system in `ui/`, swapping only the engine.

## Build

Commands live in [CONTRIBUTING.md](./CONTRIBUTING.md) → Development Setup. The cross-package rule an agent must not miss:

> editor imports `@docen/docx` by package name (→ `dist`), so **docx src changes need `pnpm --filter @docen/docx build`** before they show in the editor demo. editor/src is HMR'd — no build needed. After deleting or moving files, clear the vite dep cache (`node_modules/.vite` in the root and `packages/editor`) or the demo keeps loading ghosts.

## Data Model

One document, three projections, each owned by exactly one layer:

| Projection         | Format                                | Owner                       |
| ------------------ | ------------------------------------- | --------------------------- |
| Canonical model    | `DocumentOptions` (@office-open/docx) | file I/O, format conversion |
| Text editing       | Tiptap JSON (DOCX-rich attrs)         | editor transactions         |
| Rendering geometry | `LayoutDoc` (@docen/layout)           | pagination                  |
| Instantiated scene | LeaferJS elements                     | editor canvas painter       |

**Define once, pass through.** office-open's Options types are the single source of truth: Tiptap attrs mirror them verbatim (`renderDocx`/`parseDocx` are near-identity passes), and the layout projection reads the same attrs. No layer re-derives a property another layer already carries; a mapping exists once (stringify side and parse side together).

## API Layering

Standalone functions are core; extension commands are thin wrappers.

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DemoMacro/docen](https://github.com/DemoMacro/docen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
