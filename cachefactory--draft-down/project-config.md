---
trigger: always_on
description: This workspace contains a DraftDown 3D CAD application built with Electron + React + Three.js. The architecture is defined in `archigraph.yaml` — **always consult it first** for questions about how the system works.
---

# ArchiGraph Workspace — DraftDown

This workspace contains a DraftDown 3D CAD application built with Electron + React + Three.js. The architecture is defined in `archigraph.yaml` — **always consult it first** for questions about how the system works.

## Quick Reference

**"How does X work?"** → Search `archigraph.yaml` for the relevant node ID and read its `docs.description`. Follow its edges to understand connections.

**"Where is X implemented?"** → The node's `id` maps to a source file. Key mappings:
- `process.main` → `src/main/main.ts` (Electron main process)
- `process.renderer` → `src/renderer/` (React UI + Three.js)
- `app.singleton` → `src/renderer/Application.ts` (orchestrator)
- `bridge.scene` → `src/renderer/SceneBridge.ts` (geometry → Three.js sync)
- `engine.geometry` → `src/engine/geometry/GeometryEngine.ts` (B-Rep kernel)
- `data.document` → `src/data/ModelDocument.ts` (owns all state)
- `tool.*` → `implementations/tool.*/` (one folder per tool)
- `tool.base` → `implementations/tool.base/` (BaseTool + shared tool infrastructure: drawingPlanes, planeGeometry, VertexTransformSession, GripOverlay)
- `tool.manager` → `implementations/tool.manager/ToolManager.ts`
- `renderer.webgl` → `src/renderer/WebGLRenderer.ts`
- `camera.main` → `src/renderer/CameraController.ts`
- `viewport.main` → `src/renderer/Viewport.ts`
- `system.snap` → `implementations/renderer.webgl/SceneBridge.ts` (findSnapPoint method)
- `system.autoface` → `src/engine/geometry/GeometryEngine.ts` (autoCreateFaces/splitFaceWithEdge)
- `system.undo` → `src/data/HistoryManager.ts`

## Workspace Structure

```
.
├── CLAUDE.md              # This file
├── archigraph.yaml        # Architecture: 123 nodes, 547 edges — THE SOURCE OF TRUTH
├── schema.yaml            # Vocabulary: layers, node kinds, edge kinds
├── package.json           # Dependencies: electron, react, three, typescript
├── src/
│   ├── core/              # Shared types, interfaces, math utilities
│   ├── main/              # Electron main process + preload
│   ├── renderer/          # React UI, Three.js renderer, camera, viewport, scene bridge
│   ├── engine/            # Geometry engine (B-Rep), inference engine, constraints
│   ├── data/              # Document, scene, selection, history, materials managers
│   ├── tools/             # All 23 drawing/modify/navigate tools
│   ├── operations/        # Geometry operations (extrude, boolean, fillet, etc.)
│   ├── file/              # File format handlers (native, OBJ, STL, glTF, DXF)
│   ├── workers/           # Web workers for mesh processing and file I/O
│   ├── native/            # WASM bridge stubs (Manifold, OpenCascade)
│   └── plugins/           # Plugin system
├── tests/
│   └── e2e-playwright/    # Real Electron E2E tests (93+ tests, no mocks)
├── dist/                  # Built output (webpack)
│   ├── main/              # main.js + preload.js
│   └── renderer/          # renderer.js + index.html
└── implementations/       # ArchiGraph-generated requirement docs per node
```

## Key Architecture Decisions

### Application Bootstrap
`ViewportCanvas` React component creates the `Application` singleton on mount → initializes `ModelDocument`, `Viewport` (Three.js), `SceneBridge`, `InferenceEngine`, `ToolManager` (registers all 23 tools). See node `app.singleton`.

### Geometry → Rendering Pipeline
`GeometryEngine` (B-Rep half-edge mesh) → `SceneBridge.sync()` → Three.js scene objects (face groups + edge lines). See nodes `engine.geometry`, `bridge.scene`.

### Keyboard Event Architecture
ALL keyboard events go through a SINGLE `window` listener in `App.tsx`. No `onKeyDown` on the viewport container (removed to prevent double-firing with toggle-based plane switching). Routes: Cmd+Z→undo, letters→tool activation, arrows→tool plane switching, Escape/Enter/Delete→tool action. Escape is classic-CAD-style: mid-operation it cancels the operation and KEEPS the tool; with the tool idle it clears the selection and returns to Select. Arrow keys bypass the INPUT focus check. Electron menu has NO accelerators. See node `system.keyboard`.

### Tool Event Flow
Mouse event on container div → `ViewportCanvas.getToolEvent()` (raycast + snap) → `tool.onMouseMove/Down/Up(event)` → geometry changes → `app.syncScene()` + `app.syncSelection()`. See edges from `process.renderer` to `app.singleton`.

### Selection & Highlighting
Raycast tests both main scene (faces) and overlay scene (edges). Edge hit threshold scales with camera distance (2% of dist). **Edges prioritized over faces** in results when hit — makes edge selection reliable. Highlight: face materials swapped (polygonOffset -1), edge materials swapped + glow tube cylinder for visible thickness. Pre-selection = orange, selection = blue. See node `renderer.webgl`.

### Edge Rendering
Edge lines are in the **overlay scene**, rendered in a separate pass AFTER the main scene with depth cleared. Edge materials ARE swapped for highlighting now (with glow tube). Highlight restored on mouse-off. See node `bridge.scene`.

### Undo/Redo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CacheFactory/draft-down](https://github.com/CacheFactory/draft-down) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
