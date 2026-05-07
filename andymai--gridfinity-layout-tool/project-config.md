---
trigger: always_on
description: Gridfinity Layout Tool: React + TypeScript web app for 3D-printed drawer organizer layouts.
---

# CLAUDE.md

Gridfinity Layout Tool: React + TypeScript web app for 3D-printed drawer organizer layouts.

**Stack:** React 19, TypeScript 6, Vite 7, Zustand 5 + Immer, Tailwind CSS 4, Three.js, Vitest, Playwright, PWA, Vercel Blob + Redis, Liveblocks, PostHog.

## Git & Quality

- **Main is protected** - all changes via PRs
- Pre-commit hooks enforce lint-staged, module boundaries, i18n (4 checks), exhaustiveness, affected tests, component structure, missing tests, readme reminders

## Code Style (Enforced)

| Required                      | Prohibited                |
| ----------------------------- | ------------------------- |
| `import type` for types       | `any` (use `unknown`)     |
| Explicit types                | `console.log`             |
| `useShallow` for multi-select | `var`, `==`               |
| `@/` path alias               | Non-null assertions (`!`) |

## Directory Structure

```
src/
├── core/           # Infrastructure: api/, constants.ts, labs/, result/, storage/, store/, types.ts
├── features/       # Vertical slices (each has README.md): bin-designer, bin-inspector,
│                   # categories, cloud-share, command-palette, design-linking, generation,
│                   # grid-editor, inspiration-gallery, labs, layers, layout-library,
│                   # name-suggestions, onboarding, print-export, staging
├── shared/         # Cross-cutting: analytics/, components/, constants/, contexts/,
│                   # generation/, hooks/, printSettings/, types/, utils/
├── shell/          # App shell: Header/, Sidebar/, Collab/, Mobile/, Modals/,
│                   # Tablet/, layouts/, styles/
├── design-system/  # UI primitives: Button, Checkbox, Dialog, Input, Select, etc.
└── i18n/           # Localization (en, de, es, fr, nb, nl, pt-BR)
```

## Core Architecture

### Stores (`src/core/store/`)

| Store                | Purpose                                                                          |
| -------------------- | -------------------------------------------------------------------------------- |
| `layout.ts`          | Layout data (bins, layers, categories, drawer). Returns `Result<T, LayoutError>` |
| `library.ts`         | Multi-layout library, `activeLayoutId`, thumbnails                               |
| `settings.ts`        | User preferences (localStorage: `gridfinity-settings-v1`)                        |
| `history.ts`         | Undo/redo (max 100). Automatic via CQRS undo capture middleware                  |
| `selection.ts`       | Selected bins, active layer/category                                             |
| `interaction.ts`     | Current interaction, drop targets, layer view mode                               |
| `ui.ts`              | Panel visibility, sidebar state, UI toggles                                      |
| `view.ts`            | 3D preview camera, isometric snap                                                |
| `toast.ts`           | Toast notification queue                                                         |
| `labs.ts`            | Experimental feature flags                                                       |
| `halfBinMode.ts`     | Half-bin toggle state                                                            |
| `mobile.ts`          | Mobile-specific UI state                                                         |
| `layoutAnalytics.ts` | Layout statistics tracking                                                       |
| `sharedPreview.ts`   | Shared preview/embed state                                                       |

### Data Model (`src/core/types.ts`)

```
Layout → Drawer, Categories[], Layers[], Bins[], printBedSize, gridUnitMm, heightUnitMm
Bin → position (x,y), size (w,d,h), layerId, category, label, notes, customProperties?
```

### Critical Gotchas

1. **Coordinate System**: Grid (0,0) is **bottom-left**. `layers[0]` is bottom. UI reverses via `getDisplayLayers()`.
2. **Staging**: `layerId === '__staging__'` = off-grid stash. Auto-used when bins displaced.
3. **Half-Bin Mode**: 0.5 increments. Helpers: `snapToHalf()`, `snapToGrid()`, `isFractional()`. `HALF_BIN_SCALE = 2`.
4. **Multi-Layout**: Each layout stored by UUID (`gridfinity-layout-{uuid}`). Library index tracks metadata only.
5. **Wall Pattern Border Rule**: Any feature that cuts through a wall (cutouts, handles, etc.) MUST have corresponding border clipping in `wallPatternBuilder.ts`. Cutout/handle clips use `CUTOUT_BORDER_WIDTH` (1.5mm); divider junction clips use `max(CUTOUT_BORDER_WIDTH, shapeRadius)` so larger hex prisms (4u+ bins) can't bleed into divider walls. Without border clipping, hex prisms overlap the cut region producing jagged edges.

### Result Type (`src/core/result/`)

Use `Result<T, E>` for fallible operations. Import `ok`, `err`, `isOk`, `isErr` from `@/core/result`.

Error types: `LayoutError`, `ValidationError`, `StorageError`, `ApiError`. Use `getUserMessage()` for display.

### Storage (`src/core/storage/`)

**Atomic ops (preferred):** `saveLayoutWithMetadata()`, `createLayoutEntry()`, `deleteLayoutWithEntry()`, `switchActiveLayout()`

Import from `@/core/storage` (public facade).

### Interaction Types (`src/core/types.ts`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andymai/gridfinity-layout-tool](https://github.com/andymai/gridfinity-layout-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
