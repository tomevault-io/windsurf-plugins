---
trigger: always_on
description: npm run dev          # Vite dev server on port 5173
---

# FreeCut Web

## Commands

```bash
npm run dev          # Vite dev server on port 5173
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Vitest (watch mode)
npm run test:run     # Vitest (single run)
npm run routes       # Regenerate TanStack Router tree (tsr generate)
```

## Architecture

Browser-based multi-track video editor. React 19 + TypeScript + Vite.

```text
src/
├── features/              # Self-contained feature modules
│   ├── editor/            # Editor shell, toolbar, panels, stores
│   ├── timeline/          # Multi-track timeline, actions, services
│   ├── preview/           # Preview canvas, transform gizmo, scrub renderer
│   ├── player/            # Playback engine (Clock, composition)
│   ├── composition-runtime/ # Composition rendering (sequences, items, audio, transitions)
│   ├── export/            # WebCodecs export pipeline (Web Worker)
│   ├── effects/           # GPU effect system
│   ├── keyframes/         # Keyframe animation, Bezier editor, easing
│   ├── media-library/     # Media import, metadata, OPFS proxies, transcription
│   ├── project-bundle/    # Project ZIP export/import
│   ├── projects/          # Project management
│   └── settings/          # App settings
├── domain/                # Framework-agnostic domain logic
│   └── timeline/          # Transitions (engine, registry, renderers), defaults
├── infrastructure/        # Browser/storage/GPU adapters
│   ├── gpu/               # Facades for gpu-effects, gpu-transitions, gpu-compositor
│   └── storage/           # IndexedDB persistence via idb
├── lib/                   # Core libraries (import via infrastructure/ facades)
│   ├── gpu-effects/       # WebGPU effect pipeline + shader definitions
│   ├── gpu-transitions/   # WebGPU transition pipeline + shaders
│   ├── gpu-compositor/    # WebGPU blend mode compositor
│   ├── gpu-scopes/        # WebGPU waveform/vectorscope renderers
│   ├── fonts/             # Font loading
│   ├── shapes/            # Shape path generators
│   └── migrations/        # Data migration system
├── shared/                # Shared UI/state/utilities across layers
│   ├── logging/           # Structured logger, frame jitter monitor
│   ├── state/             # Zustand stores (playback, editor, selection)
│   └── utils/             # Managed workers, media utilities
├── components/ui/         # shadcn/ui components
├── app/                   # App bootstrap, providers, debug utilities
├── routes/                # TanStack Router (file-based, auto-generated routeTree)
├── config/hotkeys.ts      # Keyboard shortcut definitions
└── types/                 # Shared TypeScript types
```

## Key Patterns

- **State**: Zustand stores + Zundo for undo/redo
- **Timeline store split**: `useTimelineStore` (from `timeline-store.ts`) is a **facade** over domain stores (`items-store`, `transitions-store`, `keyframes-store`, `markers-store`, `timeline-settings-store`, `timeline-command-store`). Components use the facade with selectors; action code accesses domain stores via `.getState()` directly
- **Timeline mutations**: Action modules in `features/timeline/stores/actions/*.ts` use `execute()` wrapper from `shared.ts` for undo/redo integration. Never mutate timeline stores directly — use these actions
- **Timeline item types**: `TimelineItem` is a discriminated union on `type`: `video | audio | text | image | shape | adjustment | composition` — GIFs use `image` type, no separate gif type. Types in `src/types/timeline.ts`
- **Item positioning**: Remotion convention — `from` (start frame in project FPS) + `durationInFrames`
- **Compositions**: Pre-compositions (sub-comps) have dedicated stores (`compositions-store.ts`, `composition-navigation-store.ts`). 1-level nesting only. Actions in `composition-actions.ts`
- **Migrations**: `lib/migrations/` — versioned migrations + normalization run on every project load. Increment `CURRENT_SCHEMA_VERSION` in `types.ts` when adding new migrations
- **Routing**: TanStack Router — run `npm run routes` after adding/changing route files
- **Path alias**: `@/*` → `src/*`
- **Styling**: Tailwind CSS 4 + shadcn/ui (Radix primitives)
- **Media processing**: Mediabunny for decode, WebCodecs for export, Web Workers for heavy ops
- **Storage**: Workspace folder via File System Access API (see `infrastructure/storage/workspace-fs/`). Source of truth is a user-picked directory on disk — projects, media metadata, thumbnails, waveforms, gif frames, decoded audio, transcripts all live as plain files. `WorkspaceGate` (`src/features/workspace-gate/`) blocks app render until a workspace is granted. IndexedDB is only used for a tiny handle registry (`freecut-handles-db` v1, at `infrastructure/storage/handles-db.ts`) that stores non-serializable `FileSystem*Handle` references. Legacy `video-editor-db` is read only by the one-time migration path under `infrastructure/storage/legacy-idb/` (reader.ts + migrate.ts); consumers import from the barrel `@/infrastructure/storage` which routes everything to workspace-fs

## Code Style

- Strict TypeScript (`noUnusedLocals`, `noUnusedParameters`, `noUncheckedIndexedAccess`)
- `no-console` rule — always use `createLogger` from `src/shared/logging/logger.ts`, never raw `console.*` calls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walterlow/freecut](https://github.com/walterlow/freecut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
