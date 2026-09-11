---
trigger: always_on
description: A lightweight, zero-dependency web-based CNC GCode viewer with 2D/3D visualization. Built as **pure vanilla JavaScript** with no frameworks or external libraries. Three versions: **standalone** (local file viewing), **FluidNC** (embedded ESP32/FluidNC integration with SD card browser), and **Font Creator** (tool for creating CNC-engraved fonts).
---

# CNC GCode Viewer - AI Coding Agent Instructions

## Project Overview
A lightweight, zero-dependency web-based CNC GCode viewer with 2D/3D visualization. Built as **pure vanilla JavaScript** with no frameworks or external libraries. Three versions: **standalone** (local file viewing), **FluidNC** (embedded ESP32/FluidNC integration with SD card browser), and **Font Creator** (tool for creating CNC-engraved fonts).

**Critical constraint:** Total uncompressed size must stay under **135KB** (~45KB gzipped). Currently at ~133KB.

**Philosophy:** Every byte counts. Prioritize code size over abstractions. Inline small functions, reuse variables, use ternary operators. No frameworks, no dependencies, no polyfills.

## Architecture

### Module Structure (ES6 Classes)
All classes are **globally scoped** (no modules/imports) for single-file HTML inlining. Each file defines exactly one class:

**Core Architecture (`src/js/`):**
- `parser.js` → `GCodeParser` - Streams GCode in 50KB chunks, converts to segments with modal state tracking
- `camera.js` → `Camera` - Shared view transforms for both renderers (pan/zoom/rotate)
- `renderer2d.js` → `Renderer2D` - Canvas 2D with manual matrix math
- `renderer3d.js` → `Renderer3D` - WebGL with MVP matrix, custom shaders, depth testing
- `animator.js` → `Animator` - Frame-by-frame playback via `requestAnimationFrame`
- `controller.js` → `Controller` - Main app logic, owns parser/camera/renderers/animator

**Extensions:**
- `fluidnc-api.js` → `FluidNCAPI` - REST client for ESP32/FluidNC devices
- `fluidnc-controller.js` → `FluidNCController` - Extends `Controller`, adds SD card browser
- `font-creator-controller.js` → `FontCreatorController` - Character glyph editor with canvas drawing, arc detection, Douglas-Peucker path simplification, kerning management
- `font-creator-app.js` - App initialization for Font Creator (no class, just init code)

**Font Creator Features:**
- **Canvas-based drawing:** Click-drag to draw character strokes on 600x600 grid
- **Arc detection:** Converts freehand curves to G2/G3 arc commands (min radius 5mm)
- **Path simplification:** Douglas-Peucker algorithm with configurable tolerance
- **Font metrics:** SVG font units (1000 units/em) with ascent/descent/cap-height guides
- **Kerning editor:** Pair-based spacing adjustments (e.g., "AV": -2)
- **Text-to-GCode:** Generates CNC toolpaths with user-defined parameters (feed rate, plunge depth, etc.)
- **Font import/export:** JSON format with character strokes, metrics, and kerning data

**Data flow:** File → `GCodeParser.parseFile()` → `segments[]` → `Controller.loadSegments()` → `Renderer*.render()` → Canvas

### Build System (`build.ps1`)
PowerShell script that creates three single-file HTML distributions:
1. **FluidNC Extension** (`dist/gcodeviewer.html.gz`) - ESP32 device integration, includes: fluidnc-api, fluidnc-controller. HTML deleted after gzip.
2. **Standalone Version** (`dist/gcodeviewer.html`) - Local file viewer, includes: parser, camera, renderer2d, renderer3d, animator, controller
3. **Font Creator** (`dist/fontcreator.html`) - Font design tool, adds: font-creator-controller, font-creator-app
4. **Landing Page** (`dist/index.html`) - GitHub Pages landing page copied from `src/index.html`

**Build process:**
1. Reads HTML template from `src/gcodeviewer{-fluidnc,}.html` or `src/fontcreator.html`
2. Inlines CSS from `src/css/common.css` (+ `fluidnc.css`/`font-creator.css` if applicable) into `<style>` tags
3. Concatenates JS files in dependency order (see `$builds` array in `build.ps1`)
4. Minifies JS with Terser (3 passes: `--compress passes=3`)
5. Removes CSS/HTML comments and strips whitespace
6. Creates `.gz` versions for embedded deployment (FluidNC only)

**Run:** `.\build.ps1` (requires `npm install -g terser`)
**Output:** `dist/*.html` + `dist/*.html.gz` (check file sizes in console output)

## Key Development Patterns

### 1. Streaming Parser Architecture
GCode files are parsed in **50KB chunks** to handle large files without freezing:
```javascript
// parser.js - Never load entire file into memory
async parseFile(file, onProgress) {
    const chunkSize = 50 * 1024; // 50KB chunks
    while (offset < totalSize) {
        const chunk = await this.readChunk(file, offset, chunkSize);
        buffer += chunk;
        const lines = buffer.split('\n');
        buffer = lines.pop(); // Keep incomplete line
        // Process complete lines...
    }
}
```

### 2. Modal State Tracking
GCode parser maintains modal state (position, units, plane, tool) across lines:
- Absolute (`G90`) vs relative (`G91`) positioning
- Units: `mm` (`G21`) or `inches` (`G20`)
- Plane selection: `XY` (`G17`), `ZX` (`G18`), `YZ` (`G19`)
- Current tool number for multi-tool coloring

### 3. Adaptive Arc Tessellation
Arcs (`G2`/`G3`) are converted to line segments with segment count based on arc length and radius:
```javascript
// parser.js - More segments for larger/longer arcs
const segmentLength = 1.0; // mm per segment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeyeager65/cnc-gcode-tools](https://github.com/jeyeager65/cnc-gcode-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
