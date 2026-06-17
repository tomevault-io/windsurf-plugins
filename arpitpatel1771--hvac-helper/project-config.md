---
trigger: always_on
description: - **Framework:** React with Vite
---

# Project Instructions

- **Framework:** React with Vite
- **Styling:** Tailwind CSS only. No custom CSS.
- **Tone:** Be concise. Don't explain basic React concepts. Do explain non-obvious architectural decisions since the owner is not an experienced frontend developer.
- **Language:** JavaScript only, no TypeScript
- **Dev OS:** Windows, PowerShell. Use `;` instead of `&&` for chaining commands.
- **Code style:** Modular, readable, broken into small components. Other developers (including non-experts) need to be able to understand this code.

---

## What This App Is

**HVAC Helper** is a local-first, browser-based floor plan annotation tool for HVAC professionals. It is an alternative to plandroid.com.

The user loads a PDF floor plan, draws labeled sections/zones on top of it (rectangles or polygons), and exports a new PDF with those annotations burned in, pixel-perfectly aligned.

---

## Current Features (Do Not Break These)

- Load and view multi-page PDFs
- Auto-scale PDF to fit the workspace
- Page-by-page navigation
- Draw rectangles (click and drag)
- Draw custom polygons (click to place nodes, click start node to close)
- Move and resize any shape via bounding box (Konva Transformer)
- Sidebar: list, rename, delete sections
- Sections are page-aware — each shape belongs to a specific page
- Auto color-coding for shapes
- Export: original PDF + all annotations overlaid, with correct coordinates
- **Rotation-aware export:** handles PDFs with internal rotation metadata (0°, 90°, 180°, 270°). Annotations always land where they were drawn visually regardless of PDF rotation.

---

## Planned Features (Design With These In Mind)

Do not implement these now, but make architectural decisions that won't block them later:

- Ductwork drawing tools (lines with width, bends)
- Load calculations based on section area/volume
- Equipment placement (drag-and-drop symbols: units, vents, thermostats)
- Bill of Materials (BOM) auto-generation from drawn plan
- Cloud sync / team sharing

This means: keep shape data extensible, keep drawing tools pluggable, keep the sidebar generic enough to show different shape types.

---

## Approved Package Versions

Always use these exact versions. Do not hallucinate versions that do not exist.

```json
"dependencies": {
  "konva": "^10.2.3",
  "lucide-react": "^0.383.0",
  "pdf-lib": "^1.17.1",
  "pdfjs-dist": "^4.4.168",
  "react": "^19.0.0",
  "react-dom": "^19.0.0",
  "react-konva": "^19.0.0",
  "uuid": "^11.0.0"
}
```

**Do NOT use `react-pdf`.** It conflicts with the Konva layer approach. Use `pdfjs-dist` directly for rendering.

---

## Architecture

Three completely separate concerns. Never mix them.

```
┌──────────────────────────────────┐
│         react-konva Stage        │  drawing, selection, interaction
├──────────────────────────────────┤
│    pdfjs-dist → canvas → image   │  PDF display only
├──────────────────────────────────┤
│            pdf-lib               │  export only, never for display
└──────────────────────────────────┘
```

### PDF Display (pdfjs-dist)
- Render the PDF page to a hidden `<canvas>` using pdfjs-dist
- Convert that canvas to a data URL via `canvas.toDataURL()`
- Render that data URL as a Konva `<Image>` in the background layer of the Stage
- This gives a static visual of the PDF that Konva can sit on top of

```
<div style={{ position: 'relative' }}>
  <canvas style={{ display: 'none' }} />     // pdfjs renders here
  <Stage>
    <Layer>
      <Image />                              // PDF as background
    </Layer>
    <Layer>
      <Rect /> / <Line />                    // user drawn shapes
      <Text />                               // shape labels
      <Transformer />                        // selection/resize handles
    </Layer>
  </Stage>
</div>
```

### Shape Drawing (react-konva)
- Rectangles: mousedown → mousemove → mouseup to define bounds
- Polygons: click to place each node, click on first node to close path, use Konva `<Line closed />`
- After finishing a shape, prompt user for a name
- All shapes stored in React state

### PDF Export (pdf-lib)
- Load original PDF bytes into pdf-lib
- For each shape on each page, convert coordinates (see below) and draw onto the pdf-lib page
- ALWAYS account for PDF page rotation when exporting (see rotation section below)
- Save and trigger a browser download

---

## Coordinate System — This Is The Most Critical Part

PDF coordinate space and Konva/screen coordinate space are completely different. Getting this wrong means shapes appear in the wrong place on export. Never pass raw Konva coordinates to pdf-lib.

| | Screen / Konva | PDF (pdf-lib) |
|---|---|---|
| Origin | Top-left | Bottom-left |
| Y direction | Increases downward | Increases upward |
| Units | Pixels | Points |

### Rectangle Conversion

```js
// stageWidth/stageHeight = the rendered Konva stage size in pixels
// pdfWidth/pdfHeight = the PDF page size in points from pdf-lib page.getSize()

function rectToPdfCoords(shape, stageWidth, stageHeight, pdfWidth, pdfHeight) {
  const scaleX = pdfWidth / stageWidth;
  const scaleY = pdfHeight / stageHeight;

  return {
    x: shape.x * scaleX,
    y: pdfHeight - (shape.y + shape.height) * scaleY,  // flip Y axis
    width: shape.width * scaleX,
    height: shape.height * scaleY,
  };
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arpitpatel1771/hvac-helper](https://github.com/Arpitpatel1771/hvac-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
