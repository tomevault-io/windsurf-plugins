---
trigger: always_on
description: CascadeStudio is a browser-based parametric CAD modeling environment. Users write JavaScript
---

# CascadeStudio — Agent Development Guide

## Project Overview

CascadeStudio is a browser-based parametric CAD modeling environment. Users write JavaScript
(or OpenSCAD) in a Monaco editor; code is evaluated in a Web Worker with OpenCascade (OCCT)
compiled to WebAssembly via Emscripten. The 3D viewport uses Three.js with a matcap material.

## Quick Start

```bash
npm run build          # builds cascade-core then cascade-studio
npx http-server ./packages/cascade-studio/dist -p 8080 -c-1 --silent
npx playwright test    # 12 tests, ~40s
```

## Architecture (Monorepo)

The project is split into two npm workspace packages:

- **cascade-core** — Reusable CAD engine (no GUI deps). Worker + OpenCascade WASM + mesher.
- **cascade-studio** — Browser IDE. Three.js viewport, Monaco editor, Tweakpane GUI.

```
packages/
  cascade-core/
    src/
      engine/
        CascadeEngine.js       ← Main-thread API wrapping Worker + MessageBus
        MessageBus.js          ← Typed worker message routing
      worker/
        CascadeWorker.js       ← Web Worker entry; evaluates user code
        StandardLibrary.js     ← CAD primitives (Box, Sphere, etc.)
        StandardUtils.js       ← Caching, hashing, history tracking
        ShapeToMesh.js         ← OpenCascade → mesh triangulation (no Three.js)
        FileUtils.js           ← STEP/IGES/STL import/export
      openscad/
        OpenSCADTranspiler.js  ← OpenSCAD → CascadeStudio JS transpiler
      index.js                 ← Package entry (exports CascadeEngine, MessageBus, etc.)
    types/
      StandardLibraryIntellisense.ts
    fonts/                     ← TTF fonts for Text3D

  cascade-studio/
    src/
      main.js                  ← ESM entry point
      CascadeMain.js           ← App shell, layout (Dockview), default STARTER_CODE
      CascadeAPI.js            ← window.CascadeAPI — programmatic API for agents
      CascadeView.js           ← 3D viewport, Three.js rendering, modeling timeline
      EditorManager.js         ← Monaco editor, code evaluation, keyboard shortcuts
      ConsoleManager.js        ← Console panel, log/error capture
      GUIManager.js            ← Tweakpane GUI panel (sliders, checkboxes)
      CascadeViewHandles.js    ← 3D gizmo handle visualization
      openscad/
        OpenSCADMonaco.js      ← Monaco language support for OpenSCAD
    css/, textures/, icon/, lib/  ← Static assets

test/                          ← Playwright tests (monorepo root)
```

## Agent API (window.CascadeAPI)

Four methods — that's it:

1. `getQuickStart()` → Learn the API (call this first)
2. `runCode(code)` → Run CAD code, returns `{success, errors, logs, historySteps}`
3. `saveScreenshot(filename)` → Download 3D model screenshot (view with Read at `.playwright-mcp/filename`)
4. `setCameraAngle(azimuth, elevation)` → Rotate view (0=front, 90=right; 0=level, 90=top)

**NEVER** use `browser_take_screenshot` (captures full page UI, not the 3D model) or `browser_run_code` (use `setCameraAngle` instead).

## Playwright Testing

WebGL requires `--use-gl=angle --use-angle=swiftshader` in playwright.config.js launch args.

```javascript
await page.goto('http://localhost:8080');
await page.waitForFunction(() => window.CascadeAPI?.isReady());
await page.waitForFunction(() => !window.CascadeAPI.isWorking(), { timeout: 60000 });

// Use runCode for tests (combines setCode + evaluate + getErrors):
const result = await page.evaluate((code) => CascadeAPI.runCode(code), myCode);
expect(result.errors).toEqual([]);

// Screenshots:
await page.evaluate(() => CascadeAPI.saveScreenshot('model.png'));
// View with Read tool at .playwright-mcp/model.png
```

## CAD Modeling — Common Pitfalls

### 1. Loft() Prefers TopoDS_Wire

`Loft()` works best with wires. After transforms (Translate, Rotate), shapes become
generic `TopoDS_Shape` even if they started as wires. Loft now auto-extracts wires
with a warning, but for clearest code use `GetWire()` explicitly:

```javascript
let w1 = Circle(10, true);
let w2 = Translate([0,0,10], Circle(5, true));
Loft([GetWire(w1), GetWire(w2)]);
```

### 2. FilletEdges() Must Be Applied Before Hollowing

When filleting a solid shape, apply FilletEdges BEFORE boolean operations that
create internal geometry. After Difference/Union, the edge topology changes and
the selector may not find the edges you expect.

```javascript
// GOOD: Fillet the solid tray, then hollow it
let tray = Extrude(face, [0, 0, height]);
tray = FilletEdges(tray, 2, Edges(tray).max([0,0,1]).indices());
tray = Difference(tray, [cavity]);  // Hollow after filleting

// BAD: Fillet after hollowing — edges may not be found
let tray = Extrude(face, [0, 0, height]);
tray = Difference(tray, [cavity]);
tray = FilletEdges(tray, 2, Edges(tray).max([0,0,1]).indices());  // May fail!
```

### 3. Offset() on Faces Returns a Wire/Face, Not a Solid

`Offset(face, distance)` returns a 2D offset of the face boundary. To create a
hollow solid, offset the face and extrude separately:

```javascript
let inner = Offset(outerFace, -wallThickness);
let cavity = Translate([0, 0, wall], Extrude(inner, [0, 0, height]));
solid = Difference(solid, [cavity]);
```

### 4. Negative Volume from Face Orientation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zalo/CascadeStudio](https://github.com/zalo/CascadeStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
