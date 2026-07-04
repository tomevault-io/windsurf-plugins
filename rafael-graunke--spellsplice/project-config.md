---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start Vite dev server
npm run build     # TypeScript check + Vite production build
npm run lint      # ESLint
npm run preview   # Preview production build
npx prettier --write .  # Format all files
```

Interaction tests use Storybook play functions (`storybook/test`). When a UI bug is reported, write a failing play function to reproduce it before fixing. Portal-rendered UI (Radix context menus, dropdowns) requires `screen` not `within(canvasElement)`.

## What This Is

Spellsplice is a Magic: The Gathering video overlay editor. Users import video/audio sources, arrange clips on a non-linear timeline, and build a synchronized track of in-game events (life changes, draws, discards, etc.) that are overlaid on the exported video.

## Architecture

**App.tsx** holds all canonical state and passes it down via props. No external state management — plain React state + props drilling. Key state:
- `players: Player[]` — managed by `usePlayerTracks`; each player owns their track and events
- `selectedPlayerId: string | null` — which player's track the timeline is showing
- `selectedEvents: TrackEvent[]` — currently selected events (shown in Inspector)
- `isPlaying`, `currentTime`, `video: VideoState | null`
- `fileToLoad: File | null` — signals VideoPreview to load a video file (used after project import)
- `isDirty: boolean` — unsaved changes flag; cleared on save/import/new
- `exportDialogOpen: boolean` — controls video export dialog visibility
- Players autosave to `localStorage` under `spellsplice-autosave` on every change; restored on load.

Layout: top AppBar + left Sources panel + main area (react-resizable-panels), vertical split:
- **AppBar** — File menu (New/Open/Save/Export…/Settings). Keyboard shortcuts: Ctrl+S (save), Ctrl+O (open), Ctrl+Alt+N (new). Shows unsaved-changes confirmation dialog.
- **Sources panel** (left) — holds imported `MediaSource` files. Drag & drop or file-picker to add video/audio. Shows thumbnail + clip-use count per source. Red dot badge when any source is offline. Link icon opens `RelinkDialog` to reattach offline sources.
- **VideoPreview** (top-left, 75%) — renders video frames to a `<canvas>` via `drawImage` on a rAF loop. A hidden `<video>` element handles decoding/audio. Renders player state overlays and active windowed event banners directly on the canvas. Uses `derivedCacheRef` with a `validUntil` timestamp to skip redundant state derivation between frames.
- **Inspector** (top-right, 25%) — edits the selected event's `meta` fields. Per-type form components.
- **NLE Timeline** (bottom, 30%) — non-linear editor. Track groups per player (event rows) + shared video/audio tracks for clips. Waveform and frame thumbnails rendered on clips. Full undo/redo via `useHistory`.

## Key Types (`src/components/types/`)

- `VideoState` — `{ file, url, duration, videoEl }`
- `Card` — `{ name: string, edition?: string, revealed?: boolean }`
- `Player` — `{ id, name, lifeTotal, handSize, cards: Card[], track: Track, deckName?: string, decklist?: Decklist }`
- `Decklist` — `{ maindeck: Array<{card: Card, quantity: number}>, sideboard?: Array<{card: Card, quantity: number}> }`
- `Track` — `{ id, layers: number, events: TrackEvent[] }` — owned by a Player
- `TrackEvent` — `{ id, time, layer: number, type: EventType, resizable, duration?, meta? }` — no `color` field; color derived from `EventColorMap`
- `EventType` — 12 values: `ADD_TO_HAND`, `REMOVE_FROM_HAND`, `LOSE_LIFE`, `GAIN_LIFE`, `REVEAL_FROM_HAND`, `STACK_DECK`, `UNSTACK_DECK`, `DISPLAY_CARD`, `WIN`, `HIDE_UI`, `SHOW_UI`, `RESET`
- `MediaSource` — `{ id, name, type: 'video'|'audio', duration, file?, thumbnailUrl?, loading? }` — source file in Sources panel
- `Clip` — `{ id, type: ClipType, time, duration, sourceId, sourceOffset, trackId? }` — placed on NLE video/audio tracks. `time` = output-timeline position; `sourceOffset` = start within source file.
- `ClipType` — `VIDEO | AUDIO`
- `NLETrackGroup` — `{ id, label, type: TrackType, tracks: NLETrack[] }` — one group per player (Event type) + shared video/audio groups
- `NLETrack` — `{ id, type, events, clips?, player?, isBlocked, isHidden?, isMuted?, eventLayer? }` — single row. `eventLayer` stable index for filtering player events.
- `TrackType` — `EVENT | VIDEO | AUDIO`
- `ProjectConfig` — `{ title, author, defaultLifeTotal, defaultLayerCount, overlayStartHidden }` — project-level settings stored in state

### Event categories

- **Persistent events** (`resizable: false`, e.g. LOSE_LIFE, ADD_TO_HAND) — fire at a single point in time and permanently modify player state going forward. Rendered as icons on the track.
- **Windowed events** (`resizable: true`, e.g. DISPLAY_CARD) — span a duration and show a transient overlay while active. Rendered as bars.

`EventColorMap` in `event.ts` maps each `EventType` to Tailwind color classes (text, bg, fill, stroke).

### meta field by event type

- `GAIN_LIFE` / `LOSE_LIFE` — `{ amount: number }`
- `ADD_TO_HAND` / `STACK_DECK` — `{ cards: Card[] }` — free-text card autocomplete

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rafael-graunke/spellsplice](https://github.com/rafael-graunke/spellsplice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
