---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Cursor, Codex, etc.) working with this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Codex, etc.) working with this repository.

## What this project is

`canvas-editor-pdf` is a PDF exporter for [@hufe921/canvas-editor](https://github.com/Hufe921/canvas-editor). It re-implements the editor's rendering pipeline against [jsPDF](https://github.com/parallax/jsPDF)'s `Context2d` (a canvas-like API that emits PDF instructions) instead of an `HTMLCanvasElement`.

The library is published to npm as `canvas-editor-pdf`. `@hufe921/canvas-editor` is a peer dependency — consumers install both. There is no demo / playground — this is a library-only project.

## Commands

| Command | What it does |
|---|---|
| `npm run dev` | `vite build --mode lib-browser --watch` → rebuilds the browser bundle in `dist/` on every save. Use this when testing the library via `npm link` from a consumer. |
| `npm run build` | Full build: lint → browser bundle → Node ESM bundle → Node CJS bundle → `tsc` (types in `dist/types/`). |
| `npm run build:browser` | `vite build --mode lib-browser` → `dist/canvas-editor-pdf.{es,umd}.js` |
| `npm run build:node:esm` | `vite build --mode lib-node` → `dist/node/index.es.js` |
| `npm run build:node:cjs` | `vite build --mode lib-node-cjs` → `dist/node/index.cjs.js` |
| `npm run build:types` | `tsc -p tsconfig.json` → `.d.ts` files in `dist/types/` |
| `npm run lint` | ESLint over the repo |
| `npm run cypress:open` / `npm run cypress:run` | E2E tests (Cypress) — **note:** the test files were inherited from canvas-editor and currently target a demo URL / import paths that no longer exist. Treat as broken until rewritten. |
| `node scripts/smoke/node/run.mjs` | Manual Node smoke test (renders the shared fixture to `scripts/smoke/out/out-node.pdf`). Requires a prior `npm run build`. |
| `npx serve scripts/smoke/browser` | Serve the browser smoke test page (open in browser, click "Generate PDF"). |

## Testing

Right now the only tests are the **manual smoke scripts** under
[scripts/smoke/](scripts/smoke/) (Cypress is inherited from upstream and
broken — treat as non-functional). Roadmap for automated coverage, cheapest
first; each level is independent:

1. **Unit tests for the platform shim** (Vitest — the project already uses
   Vite). Test each `IPlatform` method in isolation, run twice (browser shim
   under JSDOM/`@vitest/browser`, Node shim natively): `createMeasurementCanvas()`
   returns a working `measureText`; `loadFontAsBase64()` returns non-empty
   base64; `svgToPngDataUrl()` returns a `data:image/png;base64,…`. → `tests/platform/`
2. **PDF output assertions** (medium). Render the same fixture in browser and
   Node, then use `pdf-parse`/`pdfjs-dist` to compare *extracted text per page*
   (deterministic) rather than raw bytes — also check page count and presence
   of image XObjects. → `tests/integration/`
3. **Visual diff with tolerance** (advanced). Rasterize each PDF page to PNG
   (`pdfjs-dist` or `@napi-rs/canvas`) and compare against a baseline with
   `pixelmatch` at a threshold (~5%). Save diffs for human inspection. → `tests/visual/`
4. **Property-based fuzzing** (bonus). `fast-check` to generate random
   `IEditorData` and assert `render()` never throws.

Suggested priority: shim units next (high value, cheap), then output
assertions, then visual/fuzzing on demand once subtle rendering bugs appear.

## Reducing published font size

`public/font/` ships full TTFs; `msyh.ttf` / `msyh-bold.ttf` (CJK) alone are
~35 MB of the ~42 MB package. [scripts/subset-fonts.md](scripts/subset-fonts.md)
documents how to subset them with `pyftsubset` before a release. It's optional
and **not** wired into `npm run build`.

## Consumer-facing examples

[examples/](examples/) holds copy-pasteable code snippets for the most common
consumer scenarios (Next.js Pages Router, Next.js App Router, Express,
standalone Node script, browser frontend). When the public API changes —
constructor signature, `fontSource` option, peer deps — update those files so
README links don't drift out of sync.

Release flow: `scripts/release.js` validates `dist/` exists, strips `dependencies` from `package.json`, runs `npm publish`, then restores `package.json`.

## Big-picture architecture

### Entry point is dual-purpose

[src/index.ts](src/index.ts) is **both** the library entry (`vite.config.ts` mode `lib`) and the demo script (`index.html` references `/src/index.ts`). It exports `DrawPdf` and the `svgString2Image` helper plus a couple of types.

### `DrawPdf` is the whole library

[src/core/draw/DrawPdf.ts](src/core/draw/DrawPdf.ts) is the orchestrator. The consumer flow is:

```js
const instance = new DrawPdf(options, data, { loadDefaultFonts: true })
await instance.setValue(data)   // optional, async because LaTeX SVG→PNG
instance.render()                // builds the jsPDF document in memory
instance.getPdf().save('x.pdf')  // download
```

`DrawPdf` instantiates and wires together:

- **Frames** ([src/core/draw/frame/](src/core/draw/frame/)) — full-page chrome drawn per page: `Background`, `Margin`, `Header`, `Footer`, `PageNumber`, `LineNumber`, `Watermark`, `PageBorder`, `Placeholder`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [douglasmatheus/canvas-editor-pdf](https://github.com/douglasmatheus/canvas-editor-pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
