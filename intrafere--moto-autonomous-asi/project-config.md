---
trigger: always_on
description: **DOMPurify sanitization MUST be active in LatexRenderer.jsx.** Processes untrusted LLM output — XSS risk without it.
---

# LaTeX Renderer System

## 🔒 CRITICAL SECURITY REQUIREMENTS

**DOMPurify sanitization MUST be active in LatexRenderer.jsx.** Processes untrusted LLM output — XSS risk without it.

**Required implementation:**
- `import DOMPurify from 'dompurify';` in LatexRenderer.jsx
- `DOMPURIFY_CONFIG` constant defined (see below)
- `renderedHtmlSmall` useMemo (single chunk) and per-chunk `renderedHtml` useMemo both call `DOMPurify.sanitize()` AFTER `renderLatexToHtml()`
- `dangerouslySetInnerHTML` receives ONLY sanitized HTML

### DOMPurify Configuration (REQUIRED)

```javascript
const DOMPURIFY_CONFIG = {
  ALLOWED_TAGS: [
    'div', 'span', 'p', 'br', 'hr', 'strong', 'b', 'em', 'i', 'u', 's', 'sub', 'sup', 'small',
    'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'ul', 'ol', 'li', 'dl', 'dt', 'dd',
    'table', 'thead', 'tbody', 'tr', 'th', 'td',
    'math', 'semantics', 'mrow', 'mi', 'mo', 'mn', 'msup', 'msub', 'mfrac', 'mroot', 
    'msqrt', 'mtext', 'mspace', 'mtable', 'mtr', 'mtd', 'annotation', 'annotation-xml',
    'svg', 'path', 'line', 'rect', 'circle', 'g', 'use', 'defs', 'clippath',
  ],
  ALLOWED_ATTR: [
    'class', 'id', 'title', 'style',
    'mathvariant', 'encoding', 'xmlns', 'displaystyle', 'scriptlevel',
    'columnalign', 'rowalign', 'columnspacing', 'rowspacing', 'stretchy',
    'symmetric', 'fence', 'separator', 'lspace', 'rspace', 'accent',
    'accentunder', 'movablelimits', 'minsize', 'maxsize', 'width', 'height',
    'd', 'viewBox', 'preserveAspectRatio', 'fill', 'stroke', 'stroke-width',
    'transform', 'x', 'y', 'dx', 'dy', 'x1', 'y1', 'x2', 'y2', 'r', 'cx', 'cy',
    'href', 'xlink:href', 'clip-path',
  ],
  ALLOW_DATA_ATTR: false,
  ALLOW_ARIA_ATTR: false,
  FORBID_TAGS: ['script', 'iframe', 'object', 'embed', 'form', 'input', 'button', 'textarea', 'select', 'option', 'link', 'style', 'base', 'meta'],
  FORBID_ATTR: ['onerror', 'onclick', 'onload', 'onmouseover', 'onfocus', 'onblur', 'onchange', 'onsubmit', 'onkeydown', 'onkeyup', 'onmousedown', 'onmouseup'],
  SANITIZE_DOM: true,
};
```

### Sanitization Point (REQUIRED)

DOMPurify sanitization occurs in two paths:
- **Small documents** (single chunk): via `renderedHtmlSmall` useMemo in main component
- **Chunked documents**: via per-chunk `renderedHtml` useMemo in each `RenderedChunk`

Both paths call `DOMPurify.sanitize(rawHtml, DOMPURIFY_CONFIG)` after `renderLatexToHtml()`.

---

## Overview

Dual rendering: **Rendered LaTeX View** (KaTeX math, dark theme on screen, white for PDFs) and **Raw Text View** (plain text, dark theme). All rendering flows through `LatexRenderer.jsx` (`frontend/src/components/LatexRenderer.jsx`). CSS in `LatexRenderer.css`. PDF generation via `downloadHelpers.js`.

**Performance Architecture**: For large documents (>~10K words), content is split into chunks at section boundaries. Each chunk renders independently via `IntersectionObserver`-gated `RenderedChunk` components — only chunks visible in the viewport (plus 600px margin) are LaTeX-rendered. Documents >50K chars auto-default to raw mode with a banner to switch. Content updates in rendered mode are debounced (1.5s) to prevent rapid re-rendering.

### Props API

```javascript
<LatexRenderer
  content={string}       // Raw content to render
  className={string}     // Optional CSS class
  showToggle={boolean}   // Show Rendered/Raw toggle (default: true)
  defaultRaw={boolean}   // Start in raw mode (default: false)
  showLatex={boolean}    // External view mode control (optional)
/>
```

---

## Rendering Pipeline (CRITICAL ORDER)

Must execute in this exact order in `renderLatexToHtml()`:

1. **`decodeHtmlEntities()`** — FIRST
2. **`autoWrapMath()`** — Auto-wrap unwrapped math
3. **`processTheoremEnvironments()`** — TikZ handling happens HERE (all three patterns: `\[...\]`, `$$...$$`, standalone)
4. **`replaceSectionCommand()`** — Section headers
5. **Text formatting, citations, footnotes, lists, tables, QED symbols**
6. **KaTeX rendering** via `renderKatexSafely()` — `maxExpand: 5000`, skips HTML placeholder content
7. **Line breaks/horizontal rules** (`\\` → `<br/>`, `\hrule` → `<hr/>`) — AFTER KaTeX
8. **DOMPurify sanitization** — LAST

**Critical:** `\\` line break conversion MUST be after KaTeX (valid syntax in `aligned`, `matrix`, etc.)

### Chunked Rendering Architecture

**Chunking**: `splitIntoChunks()` splits content at section headers and double-newlines. Target chunk size ~3000 chars. Documents under target stay as single chunk (no overhead).

**Virtualization**: Each chunk is a memoized `RenderedChunk` component. An `IntersectionObserver` (600px root margin) triggers LaTeX rendering only when the chunk scrolls near the viewport. Off-screen chunks show a height-estimated placeholder div.

**Debouncing**: `useDebouncedValue` hook delays rendered-mode processing by 1.5s during rapid content updates (WebSocket + polling). Raw mode is unaffected (instant updates).

**Auto-threshold**: Documents >50K chars (`LARGE_DOC_THRESHOLD`) auto-default to raw mode with a banner offering to switch to rendered view.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intrafere/MOTO-Autonomous-ASI](https://github.com/Intrafere/MOTO-Autonomous-ASI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
