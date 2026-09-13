---
trigger: always_on
description: > Read this at the start of every session. This is a living document — update it when decisions change.
---

# AGENTS.md — Citadel Project Context

> Read this at the start of every session. This is a living document — update it when decisions change.

---

## Project Identity

- **Name:** Citadel
- **Type:** Electron desktop app (Windows)
- **Purpose:** Atmospheric archive for memory, research, reference, introspection, and nonlinear thought — an infinite canvas where files, notes, media, and ideas become relics that can be marked, arranged, annotated, searched, and bound together.
- **Theme:** Clean, dark archival workspace. Clear typography, restrained contrast, and readable icons; no fantasy/gothic/arcane framing or themed cursors. Purposeful, Refflow-like motion is welcome when it improves orientation or feedback and respects reduced motion.
- **File extensions:** `.citadel` (JSON project), `.citadelz` (zip archive with bundled assets)
- **License:** MIT. Open source from day one.

## Defining Movement

Citadel is **not primarily a worldbuilding application**. It may support worldbuilding, campaign planning, game design, visual development, personal archives, academic research, grief/memory work, and creative investigation, but no single use case should dominate the default product language.

Core direction:

> Citadel is a dark archival canvas for collecting files, memories, research, fragments, and thoughts, then marking, arranging, and binding them into visible patterns.

Preferred product language — **for how the product thinks, not for what the interface says.**
Controls use plain words; see [UI Vocabulary](./docs/citadel-ui-vocabulary.md), which is the
authority for any string a user reads. The concepts below stay:

- **Relic:** any imported file, capture, note, memory, reference, PDF, audio, video, model, or fragment.
- **Inscription:** an annotation, note, comment, or written mark.
- **Thread:** a visual and logical relationship between relics or thoughts.
- **Sigil:** a searchable tag, marker, category, mood, or conceptual mark.
- **Chamber:** a board or archive space.
- **Index:** the searchable catalogue across relics, inscriptions, threads, sigils, and chambers.
- **Binding:** the act of creating a meaningful thread.

Avoid making default first-class concepts like Character, Place, Clue, Event, or Faction. Those can exist as user-created sigils/templates, but the foundation is archival, reflective, and research-oriented.

See `docs/citadel-ui-vocabulary.md` for the plain-word interface map, `docs/citadel-defining-movement.md` for the full direction and `docs/citadel-performance-roadmap.md` for the medium-term performance strategy.

Current implementation decisions and the active next-step queue live in `docs/citadel-performance-roadmap.md` under **Current Decisions From Recent Sessions**. Large-chamber profiling notes live beside it in `docs/citadel-large-chamber-profile-*.md`.

---

## Stack — Use These. Don't Substitute.

| Concern | Library |
|---|---|
| Desktop shell | Electron |
| UI | React + TypeScript |
| Canvas | Konva.js + react-konva |
| Arrows/overlays | React SVG (not Konva) |
| 3D viewer | Three.js (DOM layer canvas) |
| GIF playback | gifler |
| State | Zustand (4 slices — see below) |
| Undo/Recording | Custom event log (shared system) |
| PDF export | jsPDF + html2canvas |
| Zip | JSZip |
| Styling | CSS Variables + Tailwind |
| Build | electron-vite + Vite |
| Package | electron-builder (NSIS + portable .exe) |
| Tests | Vitest + Playwright |

---

## Folder Structure

```
src/
  main/
    index.ts              ← Electron bootstrap
    ipc.ts                ← All IPC handlers (file, export, settings)
    menu.ts               ← Native app menu
    autoUpdater.ts
    crashRecovery.ts
  renderer/
    App.tsx
    store/
      canvasStore.ts      ← items, boards, connections, selection
      historyStore.ts     ← event log, undo/redo, recording sessions
      uiStore.ts          ← toolMode, theme, panels, search
    canvas/
      CanvasStage.tsx     ← Konva Stage, pan/zoom
      ItemRenderer.tsx    ← routes item.type → component
      items/
        ImageItem.tsx
        GifItem.tsx
        VideoItem.tsx
        YouTubeItem.tsx   ← Electron <webview>
        AudioItem.tsx     ← waveform + controls
        Model3DItem.tsx   ← Three.js DOM canvas
        StickyItem.tsx
        TextItem.tsx
        SwatchItem.tsx
        ComparisonItem.tsx
      overlays/
        ConnectionLayer.tsx   ← SVG bezier arrows
        SnapGuides.tsx
        SelectionBox.tsx
        LassoOverlay.tsx
      snapping/
        snapEngine.ts
        spatialIndex.ts       ← grid bucketing for perf
        alignmentGuides.ts
    ui/
      Toolbar.tsx
      ContextMenu.tsx
      BoardTabs.tsx
      Minimap.tsx
      RecordingBar.tsx
      TagSearch.tsx
      panels/
        ItemProperties.tsx
        ConnectionProperties.tsx
        KeybindSettings.tsx
    export/
      pdfExport.ts
      imageExport.ts
      zipExport.ts
    plugins/
      pluginRegistry.ts
      pluginAPI.ts
      hooks.ts
    keybinds/
      actions.ts            ← all named ActionName strings
      defaultKeybinds.ts
      keybindResolver.ts    ← keydown → ActionName → handler
    theme/
      ThemeProvider.tsx
      dark.css
      light.css
```

---

## Core Data Types

### CanvasItem
```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kannibalk1w1/Citadel](https://github.com/kannibalk1w1/Citadel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
