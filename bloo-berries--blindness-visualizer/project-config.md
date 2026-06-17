---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
npm start          # Start development server (localhost:3000)
npm run build      # Production build (prebuild generates llms + OG images)
npm run build:prod # Production build without sourcemaps
npm test           # Run tests in watch mode
npm test -- --watchAll=false   # Run tests once
npm run clean      # Remove build/ and node_modules/.cache/
npm run generate:llms  # Generate LLM data files
npm run generate:og    # Generate Open Graph images for famous people
npm run build:analyze  # Build and open webpack bundle analyzer
```

**CI note**: CI sets `CI=true`, which treats ESLint warnings as errors. Suppress intentional console statements with `// eslint-disable-next-line no-console`. CI uses Node 18.x, runs `npm ci`, build, and tests (with `--passWithNoTests`). Tests live in `src/__tests__/`.

## Architecture Overview

This is a React 18 + TypeScript application built with Create React App that simulates various vision conditions in real-time. It uses Three.js for WebGL-based visual effects processing.

**At a glance**: 209 famous people, 144 vision condition types, 27 animated effects, 26 languages, 9 pages.

### Core Data Flow

1. **Input Sources** (`InputSource` type in `src/types/visualEffects.ts`): uploaded image or YouTube video (webcam is disabled, forced to YouTube)
2. **Effects System**: Effects are defined in `src/data/effects/` and combined in `src/data/visualEffects.ts`
3. **Rendering Pipeline**: The `Visualizer` component processes input through multiple layers (see below)

### Simulator Flow (2-Step)

The `VisionSimulator.tsx` component has a 2-step flow (no MUI Stepper UI):

| Step | Content | Notes |
|------|---------|-------|
| 0 | **Input Selection** — `InputSelector` for choosing YouTube video or image | Auto-advances to step 1 after 300ms when source is selected |
| 1 | **Conditions & Live Simulation** — `ControlPanel` with embedded `Visualizer` | Users toggle conditions and see effects on live video in real-time |

The `ControlPanel` accepts a `visualizerSlot: React.ReactNode` prop — the `Visualizer` component is passed in as a slot and rendered alongside the effects list. The container widens to `1400px` on step 1 to accommodate the side-by-side layout.

**Note**: The `GuidedTour` component is active — imported and rendered in `VisionSimulator.tsx` (hidden in famous-people preconfigured mode). The component file is at `src/components/GuidedTour.tsx`.

### Multi-Layer Rendering System

The rendering pipeline uses multiple techniques simultaneously, each handling different effect types:

| Layer | File | Purpose |
|-------|------|---------|
| WebGL Shaders | `shaders/` directory | Color blindness matrix transformations (for canvas-based rendering) |
| CSS Filters + DOM-injected SVG | `colorVisionFilters/`, `cssFilterManager.ts` | Color vision simulation (DOM-injected SVG filters), blur, contrast, person-specific filters (29 custom filter files) |
| DOM Overlays | `overlayManager.ts` | Visual field loss, scotomas, floaters (19 custom person overlays) |
| Animated Overlays | `hooks/animatedOverlays/` | JS-driven animated effects (21 animation files) |

**Color vision filter approach**: The `colorVisionFilters/` directory is split into focused modules: `mobileDetection.ts` (device detection + CSS fallbacks), `colorVisionMatrices.ts` (Machado 2009 matrices + interpolation), `domSvgManager.ts` (SVG container/filter injection/cleanup), and `index.ts` (barrel re-exports + main `getColorVisionFilter`/`getColorVisionFilterData`/metadata functions). `getColorVisionFilter()` injects `<filter>` elements with `<feColorMatrix>` into a hidden `<svg id="cvd-svg-filters">` container in `document.body`, returning `url("#cvd-{type}")` references. This DOM-injection approach replaced the earlier data URI method because Safari/WebKit does not support `filter: url("data:image/svg+xml,...")` (WebKit Bug #104169). The Machado 2009 matrices are blended with identity based on intensity. Monochromacy uses CSS `saturate()/contrast()` instead of SVG. `cleanupAllDOMFilters()` removes all injected elements.

Overlay z-index hierarchy is defined in `overlayConstants.ts` — new overlays must respect this ordering:
- `Z_INDEX.VISUAL_FIELD_LOSS`: 9000 (top)
- `Z_INDEX.BASE`: 5000 (default overlays)
- `Z_INDEX.DIPLOPIA`: 5001
- `Z_INDEX.VISUAL_DISTURBANCE`: 4000
- `Z_INDEX.ANIMATED`: 10 (relative)
- `Z_INDEX.ANIMATED_VISUAL_FIELD_LOSS`: 110

**Important**: CSS filters on the parent container also affect animated overlay children. When creating dark-themed animated overlays (e.g., Fujitora, Julia Carpenter), let the overlay itself provide darkness via its base gradient and keep the CSS filter brightness moderate (45%+). Crushing brightness below ~15% in the CSS filter will make overlays invisible.

### Visualizer Hooks (`src/components/Visualizer/hooks/`)

The Visualizer component uses modular hooks:
- `useAnimatedOverlay` - Visual Aura, CBS Hallucinations, Blue Field, PPVP, Palinopsia, Starbursting, and person-specific animated effects (21 individual animation files in `hooks/animatedOverlays/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloo-berries/blindness-visualizer](https://github.com/bloo-berries/blindness-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
