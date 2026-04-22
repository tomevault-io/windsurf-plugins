---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**socratink** is an MVP-stage hosted learning product, not just a standalone prototype. This repo contains both the deployable app and the `learnops/` prompt assets that drive extraction and drill behavior.

Canonical repo identity, workflow ownership, and agent constraints live in `AGENTS.md` and `docs/project/state.md`. This file is implementation-oriented and should stay focused on runtime shape and local development details.

Current product framing: AI should remove prep friction and increase truthful retrieval reps without replacing the learner's generation step.

The UI maps internal JS state names to visual indicators (colored `.concept-dot` elements in the sidebar list, plus dark mode for `hibernating`):
- `growing` — active/structurally sound
- `fractured` — misconception detected, needs repair
- `hibernating` — consolidating; triggers `body.night` dark mode
- `actualized` — consolidated/mastered

> Note: `instantiated` still exists in the state machine for backward compatibility with old localStorage data, but new concepts are never created in that state.

## Running the Project

No build step. The FastAPI backend serves frontend files from `public/`:

```bash
uvicorn main:app --reload
# then open http://localhost:8000
```

## Architecture

The app is pure vanilla HTML/CSS/JavaScript with no build step and no frameworks. Frontend lives in `public/`, backend at root.

| File | Role |
|------|------|
| `public/index.html` | Shell: HTML structure + script/style tags |
| `public/styles.css` | CSS entry — `@import`s from `public/css/` |
| `public/js/app.js` | Main `App` IIFE — all UI logic, state machine, data store |
| `public/js/graph-view.js` | Cytoscape-based knowledge graph; derives `locked` / `drilled` / `solidified` node states from the knowledge map |
| `public/js/ai_service.js` | `AIService.generateKnowledgeMap()` — calls Python backend |
| `public/js/store.js` | localStorage CRUD, concept state definitions, transient content store |
| `public/js/bus.js` | Pub/sub event emitter for cross-section communication |
| `public/js/dom.js` | DOM element references |
| `public/js/geo.js` | Crystal polygon coordinates + easing utilities |
| `public/js/morph.js` | Crystal shape morphing engine (requestAnimationFrame) |
| `main.py` | FastAPI backend — `/api/extract`, `/api/drill`, static file serving |
| `ai_service.py` | Gemini API calls for extraction and Socratic drill |
| `api/index.py` | Vercel serverless entry point (imports `main.py`) |

**Script load order in `index.html`:** `ai_service.js` (global) → `app.js` (ES module)

**Key patterns in `app.js`:**
- **`setState(state)`** — central function that drives all UI transitions; calls `applyControlsForState()` at its end — this is the *only* place controls are updated (no manual `showControls`/`setButtons` calls elsewhere)
- **`applyControlsForState(state, concept)`** — owns all button visibility/enabled state for a given state
- **`setButtons(extract, drill)`** — two-parameter helper (extract-enabled, drill-enabled); `btn-present` was removed
- **CSS variables** — `--bg`, `--primary`, `--danger`, `--success` etc. for theming; `hibernating` state triggers a dark mode swap via `body.night`
- **24-hour timer** — `setInterval`-based countdown; a hidden dev-skip button (top-right corner, click to reveal) fast-forwards it for demos
- **Drill UI** — `#drill-ui` panel with `#chat-history` / `#chat-input` for chat-style Socratic drilling
- **Data persistence** — `localStorage` keys `learnops_concepts` (array of concept objects) and `learnops_active` (selected concept ID); max 4 concepts
- **Knowledge map as source of truth** — drill outcomes are patched into `concept.graphData` and the graph derives visual state from `drill_status` / `gap_type`, rather than treating Cytoscape state as authoritative
- **Transient content store** — `contentStore = new Map()` (module-level) holds full text keyed by concept ID; does not survive page reload (intentional)

**`app.js` is organized into 16 numbered sections** (see the table of contents comment near line ~25):
1. DOM references
2. Pub/sub `Bus` — lightweight event emitter used for cross-section communication
3. `GEO` — polygon coordinate arrays (5 states × 8 polygons) for the SVG crystal shapes
4. `STATES` — display config (title/desc text per state)
5. Coordinate utilities + easing
6. `MorphEngine` — shared `requestAnimationFrame` loop; morphs polygon coordinates between states
7. Transition animation helpers (`playAnim`)
8. Grid rendering — builds tile + crystal SVG content
9. Data store — `localStorage` CRUD helpers + `_readFile()` shared file-reading helper
10. Drawer UI
11. Concept list render
12. CRUD — `startAddConcept()` (content-first creation form), `renderAddTrigger()`, `deleteConcept()`
13. `setState()` + `applyControlsForState()`
14. Pipeline handlers — `extract()`, `showContentOverlay()`, `hideContentOverlay()`, `drill()`, `drillPass()`, `drillFail()`, `consolidate()`, `completeConsolidation()`, `fastForward()`
15. Timer
16. Init / restore


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jon-devlapaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
