---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fretboard Visualizer is a guitar theory learning tool built with SvelteKit 2, Svelte 5, and Tailwind CSS 4. Users can click on frets to visualize notes and patterns on a guitar fretboard. The app includes a metronome, circle of fifths, scale comparer, and strum pattern builder for practice.

## Commands

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run preview      # Preview production build
npm run check        # Type-check with svelte-check
npm run lint         # Check formatting (Prettier) and lint (ESLint)
npm run format       # Auto-format with Prettier
```

## Tech Stack

- **Svelte 5** with runes (`$state`, `$derived`, `$effect`, `$props`)
- **SvelteKit 2** for routing and SSR (static adapter for GitHub Pages)
- **Tailwind CSS 4** via `@tailwindcss/vite` plugin
- **shadcn-svelte** for UI components (based on bits-ui)
- **TypeScript** throughout
- **Tone.js** for strum pattern audio playback
- **html-to-image** for PNG/SVG export

## Project Structure

```
src/
├── routes/
│   ├── +page.svelte          # Main fretboard page
│   ├── +layout.svelte        # Root layout (dark mode)
│   ├── scale-comparer/       # Scale comparison tool
│   └── strum-pattern/        # Strum pattern builder
├── lib/
│   ├── components/
│   │   ├── ui/               # shadcn-svelte components
│   │   ├── app-shell/        # AppShell layout wrapper
│   │   ├── fretboard/        # Fretboard, FretboardSettings, ShapeOverlay
│   │   ├── metronome/        # MetronomeDisplay, MetronomeSettings
│   │   ├── circle-of-fifths/ # CircleOfFifths (shared across pages)
│   │   ├── scale-comparer/   # ScaleComparerSettings
│   │   ├── strum-pattern/    # StrumPatternDisplay, BeatCell, ChordTimeline
│   │   └── side-panel/       # SidePanel (settings container)
│   ├── fretboard/            # Fretboard feature module
│   │   ├── store.svelte.ts   # Singleton store with $state (shared key/mode)
│   │   ├── types.ts          # TypeScript interfaces
│   │   ├── constants.ts      # Scales, tunings, layout dimensions
│   │   ├── music-utils.ts    # Note/scale calculations
│   │   ├── shape-utils.ts    # Shape overlay calculations
│   │   ├── color-utils.ts    # Color manipulation
│   │   ├── storage.ts        # localStorage persistence
│   │   └── index.ts          # Barrel exports
│   ├── metronome/            # Metronome feature module
│   │   ├── store.svelte.ts   # Singleton store with $state
│   │   ├── types.ts          # TypeScript interfaces
│   │   ├── constants.ts      # Tempo limits, timing
│   │   ├── audio.ts          # Web Audio API scheduling
│   │   ├── storage.ts        # localStorage persistence
│   │   └── index.ts          # Barrel exports
│   ├── strum-pattern/        # Strum pattern feature module
│   │   ├── store.svelte.ts   # Pattern state and playback
│   │   ├── types.ts          # Beat, StrumEvent, ChordSlot types
│   │   ├── constants.ts      # Pattern presets, strum cycle
│   │   ├── audio.ts          # Tone.js guitar playback
│   │   ├── storage.ts        # Pattern persistence
│   │   └── index.ts          # Barrel exports
│   └── utils.ts              # Shared utilities (cn, deepClone, etc.)
└── app.css                   # Global styles + Tailwind
```

## Architecture Patterns

### Singleton Stores with $state

Feature stores use a factory pattern returning a singleton with `$state`:

```typescript
function createFeatureStore() {
    const state = $state({
        // All reactive state in one object
        someValue: 'default',
        isLoaded: false
    });

    function someAction() {
        state.someValue = 'updated';
    }

    return {
        state,                    // Expose for reactive access
        get derivedValue() { return $derived(/*...*/); },
        someAction,
        initialize,               // Load from localStorage
    };
}

export const featureStore = createFeatureStore();
```

**Usage in components:**
```svelte
<script lang="ts">
    import { featureStore } from '$lib/feature';
    const s = $derived(featureStore.state);  // Shorthand for state access
</script>

<div>{s.someValue}</div>
```

### Component Organization

Each feature has its own folder under `src/lib/components/` with:
- Main display component (e.g., `Fretboard.svelte`)
- Settings component (e.g., `FretboardSettings.svelte`)
- `index.ts` barrel export

Components receive the store as a prop when they need to modify state:
```svelte
interface Props {
    store: typeof featureStore;
}
let { store }: Props = $props();
```

### State Persistence

- State auto-saves to localStorage via `$effect` in the store
- Presets are stored separately from current state
- History (undo/redo) uses a stack with debounced saves

### UI Components (shadcn-svelte)

Located in `src/lib/components/ui/`. Add new components with:
```bash
npx shadcn-svelte@latest add <component>
```

Configuration in `components.json` (zinc base color, dark mode).

## Key Constants

### Fretboard (`src/lib/fretboard/constants.ts`)
- `FRET_COUNT = 24` - Number of frets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebastian-ederer/fretboard-visualizer](https://github.com/sebastian-ederer/fretboard-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
