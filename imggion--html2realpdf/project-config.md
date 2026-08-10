---
trigger: always_on
description: This is the entry point for future coding agents working on `html2realpdf`.
---

# Agent Guide

This is the entry point for future coding agents working on `html2realpdf`.

Read these local docs before changing code:

- `agents-files/README.md` explains the agent documentation set.
- `agents-files/project-structure.md` describes the runtime flow, package layout, and core technologies.
- `agents-files/file-tree.md` gives a high-signal repository map.
- `agents-files/where-things-live.md` explains where new code should go.
- `agents-files/code-patterns.md` captures repo-specific coding patterns and maintainability rules.

## Repo Summary

- Zig/WebAssembly HTML-to-real-PDF renderer named `html2realpdf`; the pipeline now includes tokenizer, tolerant flat DOM, CSS cascade, flat Box Tree, HarfBuzz OpenType shaping, SheenBidi UAX #9, libunibreak UAX #14 for web/strict, layout, pagination, display list, TrueType font handling, image resources, PDF 1.7 output, and a typed npm wrapper.
- Zig version is `0.16.0`; use the 0.16 docs: https://ziglang.org/documentation/0.16.0/ and stdlib docs: https://ziglang.org/documentation/0.16.0/std/.
- Active renderer source lives in `src/`; the npm package lives in `bindings/js/`; browser and snapshot verification live in `tests/web/`; the real React-ref integration fixture lives in `tests/react/`.
- `build.zig` defines the native executable from `src/main.zig`, the package/root module from `src/root.zig`, and the wasm executable from `src/wasm.zig`; reusable parser/tree modules are exported from `src/root.zig`.
- `tests/web/` covers structural dumps plus real PDF generation, the embedded canvas viewer, complex invoice/report fixtures, download, DOM/ref rendering, SVG charts, transparent canvas resources, and the interactive `html2pdf.js` comparison.
- `tests/benchmark/` owns shared timing, download-artifact, and PDF.js content-classification helpers plus the deterministic 30-page mixed-content stress report used by both native HTML and React benchmark surfaces.
- `tests/web/e2e/` runs the browser harness and built React fixture through Playwright on Chromium, Firefox, and WebKit.
- `.github/workflows/ci.yml` only runs the complete release gate for pull requests into `main` and pushes to `main`; it never retains or uploads an npm release artifact and never publishes. `.github/workflows/prepare-npm-artifact.yml` manually builds and stores the tarball for an explicit release tag. `.github/workflows/publish-npm.yml` is a separate manual step that validates and publishes that prepared artifact. Tag pushes do not trigger any workflow.
- `src/wpt_subset_test.zig` adapts three pinned upstream Web Platform Test scenarios into renderer-native geometry assertions; `src/robustness_test.zig` owns deterministic malformed-input, allocation-exhaustion, and large-document gates.
- `docs/css-support.md` is the public, versioned CSS support contract; `src/css/properties.zig` is its machine-readable property inventory.
- `src/layout/page_geometry.zig` owns typed page boxes, page-selector cascade, and named-page sequences. `src/layout/fragmentation.zig` consumes those sequences for variable-height page boundaries, facing-page resolution, break arbitration, and block-child propagation. Block, inline, table, Flex, and Grid formatters must use that shared fragmentainer model instead of duplicating modulo arithmetic.
- `src/paged_media.zig` selects default/named/pseudo `@page` margin-box text only after pagination establishes page names, forced blank pages, and the final page count. Keep selector matching, page counters, margin-slot geometry, and generated text commands there; do not synthesize DOM boxes or consume content flow.
- Web table fragmentation measures `<tfoot>` groups before final placement, reserves their page-end extent, and repeats both `<thead>` and `<tfoot>` only on pages occupied by the table. Keep the rollback measurement scoped to table fragments, positioned descendants, and line identifiers.
- Browser snapshots must preserve which positioned inset sides were authored; computed `top`/`left` used values derived from `bottom`/`right` cannot be reinterpreted against PDF page geometry. Pagination copies fixed templates before appending repeats so array reallocation cannot drop later fixed furniture.
- `tests/baselines/0.1.0-alpha.0/` freezes deterministic PDFs, first-page Poppler PNGs, metrics, and digests from the document profile.
- Rounded box painting and clipping preserve independently resolved elliptical radii for every corner through layout, display-list commands, and native PDF Bézier paths.
- Preserve `clip_rect`, elliptical clip radii, and clip transforms through pagination and every display-list command, and isolate each PDF clip with `q`/`Q`.
- Replaced elements preserve browser-captured intrinsic dimensions. Resolve their used size from CSS width/height and `aspect-ratio`, then apply `object-fit`/`object-position` as a clipped native PDF image transform.
- Inline CSS Text support includes percentage `text-indent`, ASCII case transforms, emergency codepoint wrapping, mixed-font baseline alignment, `vertical-align`, and `word-spacing`; preserve word spacing through fragment/display-list state and PDF Type 0 `TJ` adjustments so geometry and selectable text remain aligned.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imggion/html2realpdf](https://github.com/imggion/html2realpdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
