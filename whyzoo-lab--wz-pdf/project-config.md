---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Compile Electron main (tsc) + start Vite dev server + launch Electron
npm run dev:vite     # Vite only (web browser, no Electron)
npm test             # Vitest in watch mode
npm run test:run     # Vitest single run (CI)
npx vitest run src/path/to/file.test.tsx   # Run a single test file
npm run build        # Production build (tsc + vite build)
npm run build:exe    # Build both Windows artifacts (portable first, then NSIS installer
                     # — the NSIS afterPack hook bundles the portable as viewer-template.exe)
npm run lint         # ESLint
```

`predev` automatically compiles `electron/` before `dev` runs — no need to call `electron:compile` manually in development.

## Architecture

### Dual-process Electron app

```
electron/main.ts      ← Node/Electron main process (BrowserWindow, IPC, embedded-PDF extraction)
electron/preload.ts   ← Context bridge (exposes IPC to renderer)
src/                  ← Renderer process (React app served by Vite on :5173)
```

The renderer never uses Node APIs directly. All IPC calls go through `window.electronAPI`:

| API | Description |
|---|---|
| `onOpenFile(cb)` | File path from OS open-file event, CLI arg, or `.pdf` file-association entry point |
| `onOpenPdfBytes(cb)` | PDF bytes when launched as a viewer-exe (portable build only) |
| `readFile(path)` | Read a local file via main process (avoids CORS on `http://localhost`) |
| `exportExe(pdfData)` | Save current PDF embedded into a copy of the portable exe |
| `printWindow()` | Native OS print dialog |

### Rendering pipeline

```
PDF bytes → pdfjs-dist (Worker) → HTMLCanvasElement → Konva Stage (KonvaImage)
```

Key points:
- **pdfjs worker** is loaded via a blob URL wrapper in `src/main.tsx` that polyfills `Uint8Array.prototype.toHex` and `Map.prototype.getOrInsertComputed` before importing the real worker — both methods are absent in the Electron Chromium version but required by pdfjs 5.x.
- `usePdfPage` renders each page once and stores the result in a **module-level WeakMap cache** (`pageCache`). View-mode switches (single ↔ spread ↔ grid ↔ fullscreen) do not re-render pages.
- `PdfPage` passes `pageData.canvas` directly to `<KonvaImage>` — no `toDataURL` / `new Image()` round-trip.

### Coordinate system

Annotations are stored in **PDF-point space** (independent of zoom/scale).

- `effectiveZoom = PDF_RENDER_SCALE * zoom` (render scale 1.5 × user zoom)
- Screen → stored: `toStoredCoords(x, y, effectiveZoom)` → divides by effectiveZoom
- Stored → screen: `toScreenCoords(x, y, effectiveZoom)` → multiplies by effectiveZoom
- For pdf-lib export, Y-axis is flipped: `toPdfLibY(pdfJsY, height, pageHeight)`

### State management (App.tsx)

All state lives in `App.tsx` (no external store). Key state:

| State | Purpose |
|---|---|
| `file / fileBytes` | Current PDF — `fileBytes` kept separately for export |
| `zoom` | Display zoom multiplier (0.1–3, step 0.25). Does NOT affect render cache. |
| `rotation` | Page rotation: `0 \| 90 \| 180 \| 270` degrees |
| `viewMode` | `'single' \| 'spread' \| 'grid' \| 'fullscreen'` |
| `fullscreenLayout` | `'single' \| 'spread'` — captured from `viewMode` when entering fullscreen |
| `appMode` | `'viewer' \| 'editor'` — hides annotation tools in viewer mode |
| `activeMode` | `'select' \| 'stamp' \| 'signature' \| 'watermark' \| 'pen' \| 'rectangle' \| null` |
| `pendingStamp / pendingSignature` | Image data URL awaiting placement click |
| `scrollToPage` | Target page number for programmatic scroll (cleared after use) |
| `currentPage` | Currently visible page number (1-based) |
| `isExporting` | Boolean blocking UI during export operations |
| `isPanelOpen` | Whether the left Pages panel is open |
| `isPageOperating` | Boolean blocking UI during page insert/delete/reorder operations |
| `toast` | `{ id: number; message: string } \| null` — current toast notification |

### Component tree

```
App
├── ActionBar          ← Top bar: view/zoom controls, editor tools, Reset markup button, upload/export
├── PagePanel          ← Left sidebar: thumbnail strip, multi-select, drag-reorder, add/delete pages
│                        readOnly={appMode === 'viewer'} — visible in both viewer and editor modes
├── Toast              ← Fixed bottom-center auto-dismissing notification (2500ms)
├── [hidden file input] ← Double-click empty area to open
└── main
    └── PdfViewer
        ├── (single)   → LazyPdfPage × numPages (IntersectionObserver-gated)
        ├── (spread)   → SpreadView → LazyPdfPage pairs
        ├── (grid)     → GridView → LazyPdfPage × numPages at zoom=0.3
        └── (fullscreen) → FullscreenView → PdfPage(s) (1 or 2 depending on fullscreenLayout)
```

`LazyPdfPage` wraps `PdfPage` and defers mounting the Konva Stage until the container enters the viewport (400px rootMargin). Once mounted it stays mounted — the render cache makes re-mounts cheap.

`PdfPage` = Konva `<Stage>` with three layers: background (`KonvaImage`) + `AnnotationLayer` + in-progress drawing preview layer.

### Performance notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whyzoo-lab/WZ-PDF](https://github.com/whyzoo-lab/WZ-PDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
