---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Group Explorer 3.6 is a web-based visualization app for abstract algebra (group theory). It runs entirely in the browser with no build step — JS files are served in place as ES6 modules.

## Commands

### Type Checking (Flow)
```bash
npx flow                    # run Flow type checker
npx flow check js/Foo.js    # check a specific file
```
Flow is used primarily as inline documentation — type annotations clarify intent and data shapes. It rarely catches outright errors in practice (especially around JSON, which is inherently loosely typed).

Known debt: serialized JSON uses `type` or `class` discriminator fields inconsistently across the codebase — avoid adding more variants until this is resolved.

### Build (version stamping only)
```bash
make           # stamps GIT version into HTML pages and package.json
make clean     # remove backup files
```

### Tests
Tests are browser-based (Mocha + Chai via CDN). Open in a browser served over HTTP:
- `UnitTests.html` (root) — main test runner, imports from `tests/`
- `tests/UnitTests.html` — same file, must be accessed via HTTP (not `file://`)

Individual test files: `tests/BitSet_Tests.js`, `tests/GEUtils_Tests.js`, `tests/MathUtils_Tests.js`, `tests/Group_Tests.js`, `tests/Subgroup_Tests.js`, `tests/DefiningRelations_Tests.js`

To serve locally:
```bash
python3 -m http.server 8080   # then open http://localhost:8080/UnitTests.html
```

### Flow Type Generation (from three.js TypeScript defs)
```bash
npm run build:flowtypes
```

## Architecture

### MVVM Pattern

All visualizers (CycleGraph, Multtable, CayleyDiagram, SymmetryObject) follow a layered MVVM structure:

- **Model** (`js/*Model.js`) — visualizer state + `Group` reference, serializes via `toJSON`/`fromJSON`. E.g., `CycleGraphModel.js`, `MulttableModel.js`.
- **View** (`js/*View.js`) — rendering only (canvas/WebGL). E.g., `CycleGraphView.js`, `MulttableView.js`.
- **ViewUI** (`js/*ViewUI.js`) — gesture/input handling on top of a View. E.g., `CycleGraphViewUI.js`.
- **ViewModel** — lives inside control components like `HighlightControl.js`; mediates between Model and View.
- **Controller** (`js/CycleGraph.js`, `js/Multtable.js`, etc.) — essentially page factories: they orchestrate the model, view, and control pieces into a standalone visualizer page. The underlying pieces (Model, View, HighlightControl, etc.) are the same ones `Sheet*` assembles directly, without going through the page controller.

Pub/sub between layers uses `createModelProxy` from `js/GEUtils.js`, which returns a `SubscriptionProxy` wrapping the model — views subscribe and get notified on model changes.

**The Model serves a dual role**: it is both the MVVM reactive state and the serialization DTO for the Sheet system. `Sheet.html` embeds visualizations and can open a visualizer page (e.g. `CycleGraph.html`) as an editor; the complete display state is passed back and forth as the Model's JSON. This dual role is intentional — the Sheet's representation of a visualization *is* the model state, so splitting them would add ceremony without benefit.

**`highlightControl` is an opaque plugin slot**: when `HighlightControl` wires itself to a model, it registers itself in `model.highlightControl`. The model never inspects this field — it just carries it through `toJSON`/`fromJSON`. Each control component owns its own serialization slot; the controller doesn't need to know about them. The field type is `any` in Flow, reflecting this intentional opacity.

### Sheet System

`Sheet.html` is the most mathematically significant page in the application — it is where relationships *between* groups are shown, specifically homomorphisms (structure-preserving maps). The standalone visualizer pages show properties of a single group; Sheet is where group theory at the level of morphisms lives.

**Current state:** `SheetModel.js` is mid-refactor — not yet proper MVVM. Element classes hold direct view references and actively drive them (`redraw()`, `updateTransform()`, `updateZ()`). `addElement()` creates model and view together. Module-level global `sheetElements` Map rather than a class. `SheetViewUI.js` acts as a controller but isn't named as one.

**Target MVVM architecture:**

- **Model** — owns `sheetElements: Map<id, SheetElement>`, pure data: id, position (x,y,w,h,z), element type, and for visualizer elements an opaque `visualizer` JSON blob. No view references.
- **ViewModel** — mediates model↔view; drives view additions/removals on model changes. Clear must be sequenced explicitly: ViewModel calls `view.clear()` *before* clearing the model map — the subscription notification arrives after the map is already empty, too late for the view to know what to tear down.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rayellis4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
