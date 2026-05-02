---
trigger: always_on
description: > **LCARdS**: Star Trek LCARS card system for Home Assistant - Lit-based web components with singleton architecture
---

# LCARdS AI Coding Agent Instructions

> **LCARdS**: Star Trek LCARS card system for Home Assistant - Lit-based web components with singleton architecture

---

## 🏗️ Architecture Fundamentals

### Singleton-Based Core System

LCARdS uses a **global singleton pattern** accessed via `window.lcards.core.*` for shared intelligence:

```javascript
// Core singletons (all extend BaseService)
window.lcards.core.themeManager       // Theme & design tokens
window.lcards.core.dataSourceManager  // Entity subscriptions & polling
window.lcards.core.rulesManager       // Conditional styling engine
window.lcards.core.animationManager   // Animation coordination
window.lcards.core.validationService  // Config validation
window.lcards.core.stylePresetManager // Style system & presets
window.lcards.core.animationRegistry  // Animation caching
```

**Critical Pattern**: All services extend `BaseService` which provides `updateHass()` and `ingestHass()` for HASS object propagation. Cards should NOT manage HASS distribution - the core handles it.

### Card Architecture Hierarchy

```
LitElement (Lit web component)
    ↓
LCARdSNativeCard (HA integration, shadow DOM, actions)
    ↓
    ├─→ LCARdSCard → Simple cards (Button, Chart, Slider, etc.)
    │   • Direct singleton access
    │   • Template evaluation via UnifiedTemplateEvaluator
    │   • Lifecycle: _handleFirstUpdate(), _handleHassUpdate(), _renderCard()
    │   • ⭐ Go-forward architecture for all new cards
    │
    └─→ LCARdSMSDCard → Complex multi-overlay displays
        • Advanced rendering pipeline
        • Navigation & routing
        • Multiple control/line overlays
```

**When to use which base**:
- `LCARdSCard`: Single-purpose cards (buttons, labels, single entity display)
- `LCARdSMSDCard`: Multi-overlay complex displays with routing

---

## �️ Zone System

Every card that supports text fields exposes **named zones** — rectangular regions of the SVG surface that text fields can be targeted to via `zone: <name>`.

### Key APIs (all in `LCARdSCard`)

```javascript
// Populated by _rebuildZones() every render cycle
this._zones  // Map<name, { bounds: { x, y, width, height } }>

// Called by cards at start of each render to rebuild the zone map
this._rebuildZones(width, height)
//   → calls this._calculateZones(width, height)  (subclass override)
//   → then  this._mergeUserZones(width, height)   (applies config.zones)

// For string-based SVG pipelines (button)
this._generateZoneTextMarkup(textFields)   // → SVG string with <g translate> groups
this._generateZoneDebugMarkup()             // → SVG string with debug overlay

// For DOM-based SVG pipelines (slider, elbow)
this._injectTextFieldsToElement(svgEl, w, h)
this._injectZoneDebugOverlay(svgEl)
```

### Auto-zones per card

| Card | Auto zones |
|------|------------|
| Button (preset) | `body` |
| Button (component) | Named `text_areas` from SVG def |
| Slider | `track` + `left`/`right`/`top`/`bottom` when border enabled |
| Elbow (simple, L-corner left) | `vertical_bar`, `left_bar` (alias), `horizontal_bar`, `body`, `full` |
| Elbow (simple, L-corner right) | `vertical_bar`, `right_bar` (alias), `horizontal_bar`, `body`, `full` |
| Elbow (simple, open) | `horizontal_bar`, `body`, `full` |
| Elbow (simple, contained) | `left_bar`, `right_bar`, `horizontal_bar`, `body`, `full` |
| Elbow (segmented) | `outer_*`, `inner_*` bars, `body`, `full` |
| Elbow (frame) | `top`, `bottom`, `left`, `right`, `body`, `full` |

### User-defined zones

Users can add or override zones via `config.zones`. Mixed px/percent per axis is supported — px takes precedence when both are present:

```yaml
zones:
  sidebar:
    x: 0
    y: 0
    width: 80
    height_percent: 100
```

### Debug overlay

`config.debug_zones: true` renders coloured dashed rects + zone name labels on top of the card. Toggle in the editor's **Zones → Developer Tools** section. **Remove before shipping** — the key persists in config when left on.

### Anti-patterns

❌ Don't access `this._zones` before calling `_rebuildZones` — it will be empty or stale
❌ Don't call `_calculateZones` directly — always go through `_rebuildZones` so user overrides are applied
❌ Don't embed text markup via `_processTextFields(textFields, fullWidth, fullHeight)` in a zone-aware card — use `_generateZoneTextMarkup` (button) or `_injectTextFieldsToElement` (slider/elbow) instead

---

## �🔧 Development Workflows

### Build & Test Commands

```bash
npm run build          # Production build (outputs to dist/lcards.js)
npm run build:dev      # Development build with source maps
npm run clean          # Remove dist folder
npm run analyze        # Bundle size analysis
```

**Critical**: After code changes, ALWAYS run `npm run build` before testing in Home Assistant. The card loads from `dist/lcards.js`, not source files.

### CSS Variable Governance

All CSS variable references (`--lcars-*`, `--lcards-*`) and theme token paths (`theme:`) are validated by `scripts/validate-css-vars.js` (4-pass). The build is **gated** on this validator — a violation breaks `npm run build`.

```bash
npm run validate:css-vars          # Run the validator
npm run validate:css-vars:verbose  # Show all valid refs alongside violations
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snootched/lcards](https://github.com/snootched/lcards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
