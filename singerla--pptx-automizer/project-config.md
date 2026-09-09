---
trigger: always_on
description: Guidance for AI agents (and new contributors) working on this codebase.
---

# AGENTS.md — Working on pptx-automizer

Guidance for AI agents (and new contributors) working on this codebase.
For the improvement plan derived from the 2026-08 architecture audit, see [ROADMAP.md](./ROADMAP.md).
For the end-user AI guide (how to *use* the library with an AI assistant), see [AI-INSTRUCTOR.md](./AI-INSTRUCTOR.md).

## What this library does

`pptx-automizer` is a **template-based** .pptx generator for Node.js. It does not
build presentations from scratch — it opens existing .pptx files (which are ZIP
archives of OOXML XML parts), copies slides/masters/shapes between them, and
mutates the XML through callback-based modifiers. New-from-scratch shapes are
delegated to the bundled PptxGenJS bridge.

Everything ultimately manipulates XML DOM nodes (via `@xmldom/xmldom`) inside a
ZIP archive (via `jszip`, or extracted to disk in `fs` mode).

## Commands

| Task | Command | Notes |
|---|---|---|
| Build | `yarn build` (tsc → `dist/`) | CJS only, no bundler |
| Test | `yarn test` (jest, ts-jest) | ~94 integration suites; writes real .pptx files to `__tests__/pptx-output/` |
| Single test | `npx jest __tests__/<name>.test.ts` | |
| Dev sandbox | `yarn dev` | Runs `src/dev.ts` with nodemon; scratchpad for manual testing |
| Lint | `yarn lint` | **Currently broken**: ESLint 9 is installed but config is legacy `.eslintrc.json` (flat config required). See ROADMAP. |
| Coverage | `yarn test-coverage` | **Broken glob**: `collectCoverageFrom` in `jest.config.ts` points at `.js` files |

There is **no CI** — run the test suite locally before considering a change done.

## Repository map

```
src/
  index.ts                 Public API surface: exports Automizer, `modify.*`, `read.*`, types
  automizer.ts             Facade/orchestrator class (load templates, addSlide, write/stream)
  dev.ts                   Manual dev playground (not part of the API, but currently compiled to dist/)
  classes/
    template.ts            Template = archive wrapper. Dual role: root (output) OR source template
    has-shapes.ts          Base class of Slide/Master/Layout: source/target context + deferred
                           queues; delegates the actual work to collaborators (Phase 2):
    element-importer.ts      element queue, getElementInfo/findElementOnSlide, typed dispatch
    related-content-copier.ts  copyRelatedContent (charts/images/diagrams/OLE/hyperlinks)
    slide-notes-copier.ts    notesSlide copy + number remapping
    placeholder-normalizer.ts  cleanSlide, duplicate/orphan placeholder cleanup
    content-type-registry.ts   presentation.xml slide lists + [Content_Types].xml entries
    slide.ts               Slide append logic, layout selection, placeholder merging
    master.ts, layout.ts   SlideMaster / SlideLayout import
    shape.ts               Base class for copyable shapes
  shapes/                  Chart, Image, Diagram, OLEObject, Hyperlink, GenericShape
  helper/
    xml-helper.ts          Generic XML/archive manipulation (append, removeIf, rel-id handling)
    modify-*-helper.ts     The public `modify.*` callback factories (text, table, chart, image, …)
    html-to-multitext-helper.ts / multitext-helper.ts
                           Rich-text pipeline: HTML → MultiTextParagraph[] → DrawingML
                           (known bugs + rework plan: see ROADMAP "HTML → PPTX text" track)
    xml-slide-helper.ts    Read-side slide introspection (getAllElements, dimensions, …)
    content-tracker.ts     ContentTracker: per-instance tracking of copied files/relations (used by cleanup)
    media-deduplicator.ts  Checksum index of ppt/media on the root template: an
                           identical image is copied once and shared by all relations
    archive/               IArchive impls: archive-jszip.ts (default), archive-fs.ts (debugging)
    generate/              PptxGenJS bridge for `slide.generate(...)`
  types/                   Public + internal type defs (chart-types, table-types, modify-types, …)
  interfaces/              Interfaces (IArchive, ISlide, PresTemplate vs RootPresTemplate, …)
__tests__/                 Integration tests + template .pptx files (pptx-templates/)
```

## Core execution model (important!)

Nearly everything the user calls is **deferred**. `addSlide()`, `modifyElement()`,
`addElement()` only *queue* work. The actual XML manipulation happens inside
`automizer.write()` / `.stream()` / `.getJSZip()`, which run
`finalizePresentation()`:

1. `writeMasterSlides()` — append queued masters + their layouts
2. `writeSlides()` — for each queued slide: copy slide XML + `_rels`, copy related
   content (charts/images/…), run the PptxGenJS generator, find + import/modify/remove
   queued elements, apply modification callbacks, clean unsupported tags
3. `writeMediaFiles()`, `normalizePresentation()` (slide-ID normalization, optional cleanup),
   then user-level `modify()` callbacks on `ppt/presentation.xml`

Consequences for agents:

- A bug reported "when writing" usually originates from a callback queued much
  earlier. Trace the queue (`importElements`, `modifications`, `relModifications`).
- Errors thrown inside user shape callbacks **reject `write()`** with a typed
  `CallbackError` (Phase 1 policy: fail loudly). The lenient legacy behavior —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [singerla/pptx-automizer](https://github.com/singerla/pptx-automizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
