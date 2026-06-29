---
trigger: always_on
description: A Chrome DevTools extension (MV3) for inspecting [Konva.js](https://konvajs.org/) canvas applications. Built as a **pnpm monorepo** with Turborepo, Vite 6, React 19, TypeScript 5.8, and Tailwind CSS 3.
---

# KonvaJS Devtools Inspector — Agent Guide

## Project Overview

A Chrome DevTools extension (MV3) for inspecting [Konva.js](https://konvajs.org/) canvas applications. Built as a **pnpm monorepo** with Turborepo, Vite 6, React 19, TypeScript 5.8, and Tailwind CSS 3.

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed architecture and how each feature works.

## Build & Dev

```bash
pnpm install          # Install all workspace deps
pnpm dev              # Dev mode with HMR (load dist/ in chrome://extensions)
pnpm dev:firefox      # Firefox dev mode
pnpm build            # Production build
pnpm build:firefox    # Firefox production build
pnpm zip              # Build + zip for store upload
pnpm lint             # ESLint
pnpm format           # Prettier
pnpm type-check       # TypeScript type checking
```

The built extension loads from `dist/` at the repo root.

### Example App

A local Konva test app lives at `examples/konva-app/`. Run it with:

```bash
cd examples/konva-app && pnpm install && pnpm dev
```

It provides four modes: Basic Examples, Multi Stage (with dynamic add/delete), Accessibility (a11y issue test cases), and Stress Test (~10k nodes).

## Key Technical Decisions

- **eval-based bridge**: All devtools panel ↔ page communication uses string-based `inspectedWindow.eval`. This avoids content script relay but means all page-side code must be serializable via `.toString()`.
- **Polling, not events**: The panel polls the page every 500ms for tree/selection updates rather than using Konva events or MutationObservers.
- **Module serialization**: Devtools modules are regular TypeScript functions that get `.toString()`'d and injected. They **cannot** import external modules or use closures over extension-side variables.
- **Live state reading over patching**: For features like the Animation Tracker, reading Konva's internal static state (`Konva.Animation.animations`, `Konva.Tween.attrs`) is preferred over Proxy-based constructor patching, which gets bypassed by bundlers like Vite.
- **Native Konva APIs over DOM reimplementation**: The Transform Gizmo uses `Konva.Transformer` directly on the canvas rather than custom DOM overlays, leveraging Konva's own transform math for correctness.
- **Pinned nodes in chrome.storage**: Bookmarked/pinned node IDs are persisted in `chrome.storage.local` under the key `konva_pinned_ids`.

## Code Conventions

- **React**: Functional components, hooks only. No class components.
- **Styling**: Tailwind CSS for layout + CSS custom properties (variables) for theming. Panel also uses SCSS (`Panel.scss`).
- **Theme**: Light/dark via `chrome.storage` + `useStorage(exampleThemeStorage)`, toggling `.light`/`.dark` CSS class on root.
- **Bridge calls**: Always guard with `window.__KONVA_DEVTOOLS_GLOBAL_HOOK__ &&` prefix before accessing hook properties.
- **Type definitions**: `pages/devtools-panel/src/types.ts` defines `OutlineNode`, `KonvaDevtools`, `Filter`, `FilterItem`, `SceneStats`, `EventInfo`, `CacheInfo`, `ProfilerSummary`, `ProfilerRecord`, `AnimationsSummary`, `AnimationInfo`, `TweenInfo`, `AttrDiff`.
- **Attribute catalogs**: `pages/devtools-panel/src/components/constants.ts` defines `NODE_ATTRS`, `SHAPE_ATTRS`, `SHAPE_CUSTOM_ATTRS` (per-shape attrs), `FILTER_SELECT`.

## File Map (Key Source Files)

```
pages/devtools-panel/src/
├── index.tsx                         # Entry: bridge function, connect, mount React
├── Panel.tsx                         # Main panel: tabs, virtualized tree, polling, search, theme
├── Panel.scss                        # Panel layout/theming CSS variables
├── types.ts                          # All shared types
├── devtools/
│   ├── connect.ts                    # Injects all hook modules into inspected page
│   ├── konvaDevtools.ts              # Core: resolves Konva global, stage/content accessors
│   ├── konvaDevtoolsOutline.ts       # Tree building, attr serialization, scene stats
│   ├── konvaDevtoolsSelection.ts     # Selection, activation, attrs, filters, export/import, diff
│   ├── konvaDevtoolsOverlay.ts       # DOM overlay, hit regions, heatmap, transform gizmo
│   ├── konvaDevtoolsProfiler.ts      # Performance profiler (patches Layer.draw)
│   └── konvaDevtoolsAnimations.ts    # Animation/tween tracker (reads Konva internals)
├── utils/
│   ├── flattenTree.ts                # FlatNode type, flattenTrees(), collectAllIds()
│   └── searchUtils.ts               # parseSearch(), matchesNode(), collectMatchIds()
├── components/
│   ├── Element.tsx                   # Tree row (virtualized, drag-drop, pin indicator)
│   ├── InspectedElement.tsx          # Right panel: attrs, events, cache, diff, a11y, screenshot
│   ├── Attributes.tsx                # Attribute editor (color picker, number scrubbing)
│   ├── Filters.tsx                   # Konva filter list + native CSS filters (v10)
│   ├── FilterItem.tsx                # Single filter with attribute controls
│   ├── EventListeners.tsx            # Event listener inspector
│   ├── CacheInspector.tsx            # Cache status and controls
│   ├── NodeDiff.tsx                  # Attribute change tracking (snapshot/diff)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/konva-inspector](https://github.com/maitrungduc1410/konva-inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
