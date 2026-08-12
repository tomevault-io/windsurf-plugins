---
trigger: always_on
description: AppShotDeck is a browser-only marketing screenshot composer for Play Store and App Store. No backend. Slide configs live in localStorage via Zustand persist. Screenshots live in IndexedDB. Export is DOM → PNG via html-to-image + WebGL compositing for 3D frames.
---

# CLAUDE.md — AppShotDeck

## What this is

AppShotDeck is a browser-only marketing screenshot composer for Play Store and App Store. No backend. Slide configs live in localStorage via Zustand persist. Screenshots live in IndexedDB. Export is DOM → PNG via html-to-image + WebGL compositing for 3D frames.

## Dev commands

```bash
npm run dev       # start dev server (Vite, port 5173)
npm run build     # tsc + vite build → dist/
npm run lint      # eslint
```

## Key architecture

### Frame system (`src/data/frames.ts`)

Two frame types, distinguished by whether `device3d` is present on the `FrameDef`:

- **Flat frames** (`outerRx` + optional `bezel`) — rendered via nested CSS divs in `SlideCanvas.tsx`.
- **3D frames** (`device3d: Device3DSpec`) — rendered via `Device3D.tsx` (WebGL). Body = `ExtrudeGeometry`, screen = `ShapeGeometry` with manually normalized UVs.

### SlideCanvas (`src/components/Canvas/SlideCanvas.tsx`)

- Always renders at full export resolution (e.g. 1080×1920). CSS `transform: scale()` shrinks it for preview.
- Branches on `frame.device3d` to choose flat CSS vs `<Device3D>`.
- `vbW` = viewBox width parsed from `frameViewBox` string — used to convert outerRx / bezelWidth from viewBox units to pixel units.
- `deviceScaleFactor = (slide.deviceScale ?? 100) / 100` — scales both slot dimensions uniformly.
- Portrait device Y: `Math.round((H - dSlotH) / 2) + Math.round(H * deviceOffset / 100)`. **0 = canvas center**, +30 = default layout position (below center).
- Landscape device X: same center-based formula using W. 0 = canvas center, +16 = default column position.
- Font sizes scale with canvas width: headline = `W * 0.063` (portrait) / `W * 0.036` (landscape), then multiplied by `headlineFontSize / 100`.
- Text shadow blur = `Math.round(W * 0.025)`. Color: dark bg → white glow, light bg → dark shadow (luminance from bg `from`/`color` hex).

### Device3D (`src/components/Canvas/Device3D.tsx`)

Critical details for the 3D renderer:

- `flat` prop on `<Canvas>` is **required** — sets `gl.toneMapping = NoToneMapping`. Removing it triggers ACESFilmic which darkens the screenshot color.
- ExtrudeGeometry with `depth=0.068, bevel=0.016`: body mesh at z=-(depth/2) → world z range [-0.050, +0.050]. Screen mesh must be at z > 0.050 → currently `depth/2 + bevel + 0.001 = 0.051`. **Do not move screen behind the body bevel tip** — transparent body renders after opaque screen and will overdraw it.
- `SizeEnforcer` compares `el.width !== Math.round(w * dpr)` (device pixels). Required to avoid infinite resize loop on retina.
- `preserveDrawingBuffer: true` — required so `toDataURL()` works for export.

### Export (`src/utils/export.ts`)

- For **flat frames**: `html-to-image` (`toPng`) captures the full-res DOM element directly.
- For **3D frames**: WebGL content can't be captured by html-to-image. Fix: call `webglCanvas.toDataURL()` first (before html-to-image runs), then composite it on top of the DOM PNG using a `<canvas>` + `drawImage()`. Position is derived from `getBoundingClientRect()` divided by the CSS scale factor.
- **Callouts + 3D frames**: the WebGL frame is drawn *over* the whole DOM capture, which would bury any zoom bubble overlapping the device. Fix: the bubbles are wrapped in a `[data-callout-layer]` div (`CalloutLayer.tsx`) and re-captured/re-drawn on top of the WebGL composite in `captureElement`. Flat frames don't hit this path (single DOM capture preserves stacking).
- **Critical**: the hidden export container in `App.tsx` must NOT use `visibility: hidden` — it's an inherited CSS property and makes html-to-image capture blank PNGs. Use `left: -9999px` only.
- **File naming / ordering** (`buildEntries` in `Header.tsx`): slides are numbered **per format**, zero-padded — `slide-01`, `slide-02`, … within each `<format-folder>`. The counter resets per format (`perFormatCount`), so a project mixing phone + tablet slides gets `phone/slide-01…` and `tablet-7/slide-01…` independently, not a global stride.
- **`addEntriesToZip(zip, entries, prefix?)` / `downloadZip(zip, name)`**: reusable helpers. `exportAll` wraps them for single-language export. "Export all languages" (`handleExportAll` in `Header.tsx`) builds **one** ZIP with per-language folders — `en/android/phone/…`, `es/android/phone/…` — by looping languages, `flushSync`-ing `exportLanguage`, and calling `addEntriesToZip` with the lang as prefix. Single-language exports still produce their own ZIP.

### Project save/load (`src/utils/project.ts`)

- Saves as ZIP: `config.json` (all slide settings) + `images/<id>.png` (one file per slide screenshot).
- Screenshots stored as real PNG files, not base64 in JSON.
- On load (`handleLoad` in Header.tsx): screenshots from ZIP are saved to IndexedDB immediately so they survive refreshes.

### Workspace save/load (`src/utils/workspace.ts`)

- Saves ALL projects as one ZIP: `workspace.json` + `images/{projectId}/{slideId}.png`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowjordanhappy/appshotdeck](https://github.com/nowjordanhappy/appshotdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
