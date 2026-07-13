---
trigger: always_on
description: Use this when an AI agent needs to produce **editable Word documents** from HTML. dom-docx converts semantic, inline-styled HTML into native OOXML (paragraphs, runs, lists, tables)—not screenshots or layout hacks.
---

# Agent guide: HTML → DOCX (dom-docx)

Use this when an AI agent needs to produce **editable Word documents** from HTML. dom-docx converts semantic, inline-styled HTML into native OOXML (paragraphs, runs, lists, tables)—not screenshots or layout hacks.

**Workflow:** agent writes HTML fragment → `convertHtmlToDocx(html)` → `.docx` buffer → save or attach.

```typescript
import { convertHtmlToDocx } from "./converter.js";

const docx = await convertHtmlToDocx(htmlFragment);
await writeFile("output.docx", docx);
```

Pass a **body fragment only** (no `<html>`, `<head>`, or `<body>` required). The converter wraps content in a letter-size document with 1″ margins and Arial 14pt body text.

---

## Design goal for agents

Optimize for three properties (in order):

1. **Valid, native structure** — paragraphs, list numbering, table rows/cells, hyperlinks
2. **Visual fidelity** — layout that survives Word/LibreOffice rendering
3. **Simplicity** — fewer nested wrappers, inline styles over external CSS, no JS

Think “Word-friendly semantic HTML,” not “web app layout.”

---

## What converts best

### Tier 1 — Excellent (~90–96% visual fidelity)

| Pattern | Notes |
|---------|--------|
| **Headings + paragraphs** | `<h1>`–`<h3>`, plain `<p>`, optional `color`, `font-size`, `text-align` |
| **Lists** | `<ul>`, `<ol>`, nested lists; one idea per `<li>` |
| **Simple tables** | `border="1"`, `cellpadding`, `width:100%`, header row, `text-align:right` on numeric columns |
| **Inline formatting** | `<strong>`, `<em>`, `<a href="…">`, `<code>`, `<br>` |
| **Inline highlights** | `<span style="background:#cfc">term</span>` on short phrases |
| **Blockquote** | `border-left:4px solid #333; padding-left:12px` |
| **Horizontal rule** | `<hr>` between sections |
| **Thematic breaks** | Section headings, not layout divs |

### Tier 2 — Good (~80–92%)

| Pattern | Notes |
|---------|--------|
| **Shaded banner blocks** | `<div style="background:#eaeaea;padding:10px 16px">` with headings inside |
| **Table row/cell styling** | `background`, `color` on `<tr>` or `<td>`; subtotal rows with `<strong>` |
| **Financial / KPI tables** | Label column left-aligned, numbers right-aligned; pastel row bands |
| **Flex rows/columns** | `display:flex; gap:8px` with 2–4 child divs (converted to borderless tables) |
| **Multi-side bordered boxes** | `border:1px solid #ccc; padding:12px` on a div (uses a 1×1 table wrapper when needed) |
| **Typography colors** | `color:#666` on subtitles, accent colors on deltas (`#2a9d8f`, `#e76f51`) |

### Tier 3 — Weak or unsupported — avoid

| Pattern | Why |
|---------|-----|
| **Inline SVG / Canvas / `<img>` charts** | Not rendered as vectors; use tables, describe data in text, or enable **`rasterizeInPlace`** when exporting from a live rendered page (recommended: `{ scale: 2 }` for chart quality) |
| **CSS bar charts inside cells** | `<div style="height:14px;width:80%;background:…">` in `<td>` does not port |
| **Emoji as UI icons** | Font/glyph mismatch in Word |
| **External stylesheets** | Only inline `style=""` and a few attributes (`border`, `cellpadding`, `colspan`) |
| **Grid / absolute / float layout** | No `position`, `float`, `grid-template` |
| **Multi-column CSS** | No `column-count` |
| **Web fonts** | Defaults to Arial; custom `@font-face` ignored |
| **Forms, inputs, buttons** | Not supported |
| **Deeply nested layout divs** | Prefer flat block flow: heading → p → table → p |
| **`<colgroup>` / `<col width>`** | Column widths inferred from content, not col tags |

---

## HTML style guide for agents

### Page and typography defaults

The engine assumes **US Letter**, **1″ margins**, **Arial 14px**, **line-height ~1.4**. You do not need a wrapper document; if you preview in a browser, match:

```html
<!-- Optional preview wrapper only — omit when calling convertHtmlToDocx -->
<style>
  body {
    margin: 0;
    padding: 96px;
    width: 816px;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 14px;
    line-height: 1.4;
    color: #111;
  }
</style>
```

Use **hex colors** (`#1a1a2e`, `#666`, `#f5f5f5`). Named CSS colors work when parsed, but hex is predictable.

### Block structure

Prefer a **linear document flow**:

```
h1 title
p subtitle (muted color, smaller font-size)
table | ul | blockquote
h2 section
p …
```

Each visual “section” should be a heading or table—not a tower of anonymous divs.

**Good — shaded hero:**

```html
<div style="background:#eaeaea;padding:10px 16px;margin-bottom:12px">
  <h1 style="margin:0;font-size:20px;color:#1a1a2e">Sprint 24 Retrospective</h1>
  <p style="margin:4px 0 0;color:#666;font-size:13px">Platform Team · Mar 3 – Mar 14, 2026</p>
</div>
```

**Bad — layout soup:**

```html
<div><div><div style="display:grid;grid-template-columns:1fr 1fr">…</div></div></div>
```

### Tables (data, not layout)

Use tables for **tabular data only**. Always include:

- `border="1"` (or explicit border in style) when grid lines are wanted
- `cellpadding="6"` or `8`
- `style="border-collapse:collapse;width:100%"`
- Header row with `<strong>` labels
- `text-align:right` on numeric `<td>` cells


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floodtide/dom-docx](https://github.com/floodtide/dom-docx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
