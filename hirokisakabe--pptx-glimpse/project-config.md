---
trigger: always_on
description: This file provides guidance to AI coding agents, including Codex and Claude Code, when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents, including Codex and Claude Code, when working with code in this repository.

## Project Overview

pptx-glimpse is a TypeScript library that converts PPTX slides to SVG / PNG.
Input: `Uint8Array` (Node.js `Buffer` is accepted as a subclass), Output: SVG string or PNG `Uint8Array`.

## Commands

```bash
npm run build          # Build with tsup (CJS + ESM + .d.ts)
npm run test           # Run all tests with vitest
npm run test -- packages/renderer/src/utils/emu.test.ts  # Run a single test file
npm run test:watch     # Watch mode for tests
npm run lint           # ESLint check
npm run lint:fix       # ESLint auto-fix
npm run format         # Prettier formatting
npm run format:check   # Prettier check
npm run typecheck      # Type check with tsc --noEmit
npm run render         # Test rendering with tsx scripts/test-render.ts
npm run inspect        # Inspect PPTX internal XML (e.g., npm run inspect -- file.pptx slide1)
npm run dev -- file.pptx  # Live preview dev server (auto-reload on packages/*/src/ changes)
```

CI consists of 6 jobs:

- **lint**: `knip` → `lint` → `format:check` → `typecheck` (Node 22 only, runs once)
- **test**: `test` with coverage → `build` → package verification (Node 22/24/26)
- **playwright**: Browser E2E tests and demo production verification (Node 22/24/26)
- **vrt**: Snapshot VRT (Docker-based, self-comparison)
- **libreoffice-vrt**: LibreOffice VRT for renderer regressions (generates fixtures and reference images via Docker)
- **editor-validity**: LibreOffice validity checks for editing API output (shares the libreoffice-vrt Docker image, runs independently from renderer VRT)

## Architecture

Read [`docs/architecture/overview.md`](docs/architecture/overview.md) before changing package
responsibilities, workspace dependencies, public/private package status, build entries,
externalization/bundling, the document-to-renderer adapter, or Node/browser boundaries. Update
the overview in the same change when one of those boundaries changes.

The working summary is: **PPTX binary → PptxSourceModel → computed view → core adapter →
private renderer → SVG → optional PNG**. `@pptx-glimpse/document` is the lower-level OOXML
foundation; `@pptx-glimpse/editor` builds on it; public `pptx-glimpse` orchestrates document,
editor, and rendering behavior; the demo/UI consumes public packages. Lower layers must not
depend on higher layers.

Read [`docs/editor-error-contract.md`](docs/editor-error-contract.md) before adding or changing
editor operations, operation failure codes, high-level editor error wrapping, warning transport,
or read/render/write catches in `PptxEditorSession`.

Before changing PptxSourceModel, computed-view, writer, or adapter behavior, also read the
module-level comments in `packages/document/src/source/pptx-source-model.ts`,
`packages/document/src/computed/pptx-computed-view.ts`,
`packages/document/src/writer/write-pptx.ts`, and
`packages/core/src/pptx-computed-view-renderer-adapter.ts`.

When adding or changing a `@pptx-glimpse/document` reader, computed view, from-scratch writer, existing-element edit, or round-trip preservation capability, update `packages/document/docs/feature-support.md` in the same change. Base every `S` entry on a public root API and an implementation test; use `△`, `P`, or `—` when support is constrained, preservation-only, or unverified, and keep the linked constraints/evidence current.

Read [`docs/development/type-assertions.md`](docs/development/type-assertions.md) before
adding or changing a type assertion, an `unsafe*Assertion` helper, parser/external boundary
narrowing, a branded constructor, or the ESLint assertion rules. Update the policy when the
allowed exceptions or enforcement changes; update its dated audit snapshot only when running
a deliberate assertion audit.

## Technical Constraints

- **SVG uses inline attributes only** — No CSS classes. resvg and librsvg do not correctly interpret CSS
- **`isArray` configuration in fast-xml-parser is required** — Tags such as `sp`, `pic`, `p`, `r` must be returned as arrays even for single elements (`ARRAY_TAGS` in `xml-parser.ts`)
- **EMU units & branded types** — PPTX internal coordinates use EMU (English Metric Units). Convert with `emuToPixels()`. A 16:9 slide is 9144000×5143500 EMU = 960×540 px. Model fields use branded types (`Emu`, `Pt`, `HundredthPt` in `packages/renderer/src/utils/unit-types.ts`) to prevent unit confusion at compile time. Use `asEmu()`, `asPt()`, `asHundredthPt()` to create branded values from raw numbers
- **Background fallback** — Backgrounds are resolved in order: slide → slide layout → slide master

## VRT (Visual Regression Testing)

Visual regression tests for rendering output. When modifying the parser or renderer, **always check whether VRT updates are needed**.

### Directory Structure

```
shared-fixtures/                              # Real PPTX files shared by e2e and VRT
├── real-basic-theme.pptx
└── real-product-page.pptx
vrt/
├── compare-utils.ts                          # Shared image comparison utilities
├── snapshot/                                 # Standard VRT (self-comparison, Docker-based)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hirokisakabe/pptx-glimpse](https://github.com/hirokisakabe/pptx-glimpse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
