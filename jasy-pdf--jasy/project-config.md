---
trigger: always_on
description: > **Ja**vaScript Ea**sy** **PDF** — a declarative, component-based PDF generation library in pure
---

# JasyPDF — CLAUDE.md

> **Ja**vaScript Ea**sy** **PDF** — a declarative, component-based PDF generation library in pure
> TypeScript, inspired by Flutter's widget tree. You describe a document as a tree of element
> objects (`PageElement`, `ContainerElement`, `TextElement`, `PaddingElement`, …) and the library
> lays it out and writes the raw PDF byte stream itself — no headless browser, no Java, no pdf-lib
> underneath. The low-level PDF writer is hand-rolled.

This file is the orientation map for working in this repo. Read it first.

## The dream (why this project exists)

Two goals, and they are **decoupled** — don't conflate them:

1. **A great declarative layout engine for documents** (Flutter-style components → PDF), with
   _first-class pagination_ — content that flows correctly across multiple pages: text breaks at
   lines, images move as a whole, columns balance, borders/padding survive a page break. This is the
   part Flo got ~85% working in a previous attempt and hit a wall on the last 15%.
2. **Open-source ZUGFeRD / XRechnung / Factur-X** support in pure TS/JS — the real strategic prize.
   The TS/Node ecosystem has no polished, dependency-light library that renders the human-readable
   invoice PDF **and** emits the conformant EN-16931 CII/UBL XML **and** validates it. Mustangproject
   (Java), horstoeko/zugferd (PHP), factur-x (Python) own the other ecosystems; Node is thin. This is
   the niche. Crucially, ZUGFeRD invoices are the _tamest_ document class (a table + totals + footer),
   so they do **not** require the hard 15% of pagination — and the hand-rolled byte-level writer is an
   _advantage_ for hitting PDF/A-3 conformance precisely.

**Competitive framing:** we are _not_ competing with pdf.js (a reader/parser) or pdf-lib/PDFKit
(low-level drawers with no layout engine). The real comparison is @react-pdf/renderer + Yoga. Beating
them on pagination correctness + DX is realistic. Beating Prince/WeasyPrint/LaTeX on typographic
quality (microtypography, hyphenation, bidi, floats) is **not** a goal and not needed for the target
use cases (invoices, reports, quotes, datasheets).

## Architecture

Two-phase pipeline: **layout** (synchronous, mutating, top-down) then **render** (async, produces
PDF content-stream strings). Entry point is `PDFDocument.render()`.

```
PDFDocument (abstract, user subclasses it, implements build())
  └─ build() → PDFDocumentElement
       ↓  PDFRenderer.render(documentElement)        src/lib/renderer/pdf-renderer.ts
       ├─ RendererRegistry.register(...) all element→renderer pairs
       ├─ document.calculateLayout()                 ← PASS 1: layout (recursive, mutates elements)
       └─ PDFDocumentRenderer.render(...)            ← PASS 2: build display list, then serialize
            └─ PageRenderer → element renderers → IRNode[] → PdfBackend.serialize → content stream
       → assembles objects, xref table, trailer → returns the PDF as a string
```

> "Pass 1 / Pass 2" are the two render passes inside one `render()` call. Don't confuse them with the
> **roadmap Phases** in `todo.md` (Phase 1 = IR seam, Phase 2 = kill singleton, …). Different things.

### Pass 1 — `calculateLayout(constraints, offset, ctx)`

- Defined on every element (`PDFElement.calculateLayout`). Signature: `calculateLayout(constraints:
  BoxConstraints, offset: Offset, ctx: LayoutContext): Size` — constraints (min/max w/h) flow **down**,
  the parent assigns each child its absolute `offset`, the element returns the `Size` it took **up**.
  The clean Flutter `RenderObject` contract (since Phase 4; `layout/box-constraints.ts`).
- `FlexLayoutHelper` (`utils/flex-layout.ts`) is axis-generic: it measures **and** places both `Column`
  (`VERTICAL_AXIS`) and `Row` (`HORIZONTAL_AXIS`), distributing leftover main-axis space to
  `ExpandedElement`s by `flex` and offsetting children per `main`/`cross` alignment.
- **Pagination is real** (Phase 5). A `fragment(maxHeight, width, ctx) → { fitted, remainder }` protocol
  (`layout/fragmentation.ts`, shared `packChildren`) splits content across pages: text at line boxes,
  padding/border cloned per fragment, flex containers re-packed. The page driver (`PDFDocumentRenderer`)
  loops the remainder into fresh physical pages; `header`/`footer` repeat on each.
- **The Y-flip lives at the IR→backend seam, NOT in elements** (Phase 3). Elements lay out in a top-left
  origin and are coordinate-blind; `PdfBackend.flipY(nodes, pageHeight)` flips once per page. `grep
  normalizeCoordinates src/` is empty.

### Pass 2 — render: display list → backend (the IR seam, since roadmap Phase 1)

The render pass is split at a hard seam — **the display list (IR)** — so the PDF byte writer never
sees a component:

- **Producers** (`src/lib/renderer/*`, one class per element, dispatched via `RendererRegistry` keyed
  on the element's constructor): each `render(element, objectManager)` returns an **`IRNode[]`**, not
  a string. Leaves (`TextRenderer`→`TextRun[]`, `LineRenderer`, `ImageRenderer`, `RectangleRenderer`)
  emit primitives; structural renderers (`Container`/`Expanded`/`Padding`, and `Rectangle` for its
  children) **concatenate** their children's lists. Producers still know about components and still do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasy-pdf/jasy](https://github.com/jasy-pdf/jasy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
