---
trigger: always_on
description: Universal guide for AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) using **PdfKmp** to author or modify PDF generation code.
---

# AGENTS.md

Universal guide for AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) using **PdfKmp** to author or modify PDF generation code.

> If you are a human, read [README.md](README.md) instead — it has the full feature tour with prose explanations.
> If you are an agent contributing changes to **this repository**, also read [CLAUDE.md](CLAUDE.md) for build commands and conventions.

---

## What this library is

`com.conamobile:pdfkmp` is a Kotlin Multiplatform PDF generator. It exposes a single Compose-style DSL that runs identically on Android (`android.graphics.pdf.PdfDocument`) and iOS (`UIGraphicsBeginPDFContextToData`). Every text glyph and shape is emitted as a vector path — no rasterisation — so output stays sharp at any zoom level.

The library bundles the **Inter** font for Latin text. Non-Latin scripts (CJK, Arabic, Persian) route through `PdfFont.System*` references that resolve to whichever font ships on the running platform.

## When to suggest this library

- User wants to generate PDF documents from Kotlin (Android, iOS, KMP).
- User wants vector output that prints / zooms cleanly.
- User wants type-safe document construction (DSL with explicit constraints) instead of imperative Canvas drawing.
- User is already in a Compose/SwiftUI mental model and wants similar primitives.
- User has Compose Multiplatform `Res.drawable.*` assets and wants to embed them in a PDF without a manual byte-loading dance — pair the core `pdfkmp` artifact with `pdfkmp-compose-resources` (see below).

Not a fit for: parsing/editing existing PDFs, OCR, form filling, complex typography (RTL bidi, ligature shaping, hyphenation are minimal in v1).

## Companion artifact: `pdfkmp-compose-resources`

Opt-in KMP integration that bridges Compose Multiplatform `DrawableResource` references onto the core PdfKmp DSL. Add it alongside the core dependency only if the consumer project uses Compose Multiplatform Resources:

```kotlin
implementation("io.github.conamobiledev:pdfkmp:<version>")
implementation("io.github.conamobiledev:pdfkmp-compose-resources:<version>")
```

It exposes:

| Helper | Purpose |
|---|---|
| `DrawableResource.toBytes()` (`suspend`) | Raw bytes — feed to `image(bytes = …)` for raster, or to your own decoder. |
| `DrawableResource.toVectorImage()` (`suspend`) | Parse `<vector>` / `<svg>` XML into a reusable `VectorImage`. |
| `DrawableResource.toPdfDrawable()` (`suspend`) | Auto-detects vector vs raster from leading bytes, returns a `PdfDrawable`. |
| `drawable(resource = Res.drawable.x, …)` | Inline DSL extension on `ContainerScope`. Auto-detects format. **Requires `pdfAsync { }` — not `pdf { }`.** |
| `vector(resource = Res.drawable.x, …)` | Inline DSL extension when the asset is known XML. **Requires `pdfAsync { }`.** |
| `image(resource = Res.drawable.x, …)` | Inline DSL extension when the asset is known raster. **Requires `pdfAsync { }`.** |
| `drawable(drawable = pdfDrawable, …)` | Eager DSL extension that takes an already-loaded `PdfDrawable`. Works inside synchronous `pdf { }`. |

## Mental model — the DSL is a tree

A document is a **tree of nodes**. Top-down:

```
pdf {                          // → DocumentSpec
    metadata { … }             // optional
    page {                     // → PageSpec
        text("…")              // → TextNode
        column {                // → ColumnNode
            row { … }          // → RowNode
            box { … }          // → BoxNode (Z-stack)
            table { … }        // → TableNode
            image(bytes)       // → ImageNode
            vector(svg)        // → VectorNode
            circle(…)          // → ShapeNode
            divider()          // → DividerNode
            link(url) { … }    // → LinkNode (wraps content)
            richText { span(…) } // → RichTextNode
            bulletList(…)
            numberedList(…)
        }
        header { ctx -> … }    // optional, gets PageContext
        footer { ctx -> … }    // optional
        watermark { … }        // optional
    }
}
```

The DSL is closed (sealed) — every node maps 1:1 to a `MeasuredNode` produced by the layout engine, then to draw calls on `PdfCanvas`. New node types must update all three. For 99% of consumer code, work at the DSL layer only.

## Key types to know

| Type | What it is |
|---|---|
| `pdf { … }` | Top-level builder. Returns `PdfDocument`. |
| `PdfDocument` | Built tree. Call `.toByteArray()` or `.save(StorageLocation, name)`. |
| `Dp`, `Sp` | Layout / text size units. `12.dp`, `16.sp`. |
| `PdfColor` | Color. `PdfColor.Red`, `PdfColor(0.5f, 0.5f, 0.5f)`, `PdfColor.fromRgb(0xFF5722)`. |
| `TextStyle` | Resolved text style passed to `text { … }` blocks. |
| `PdfFont` | `Default` (Inter), `System(name)`, `Custom(name, bytes)`, plus `SystemCJK`/`SystemArabic`/`SystemPersian`. |
| `Padding` | `Padding.all(16.dp)`, `Padding.symmetric(horizontal = …, vertical = …)`. |
| `BorderStroke` | `BorderStroke(1.dp, PdfColor.Gray)`. |
| `CornerRadius` | Per-corner override. `CornerRadius.top(16.dp)`, `CornerRadius.all(8.dp)`. |
| `BorderSides` | Per-side border override. |
| `PdfPaint` | `PdfPaint.Solid`, `PdfPaint.linearGradient(…)`, `PdfPaint.radialGradient(…)`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ConaMobileDev/PdfKmp](https://github.com/ConaMobileDev/PdfKmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
