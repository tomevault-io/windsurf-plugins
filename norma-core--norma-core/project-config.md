---
trigger: always_on
description: Guidelines for AI coding agents operating in this React/TypeScript project.
---

# AGENTS.md

Guidelines for AI coding agents operating in this React/TypeScript project.

## Commands

```bash
npm run dev          # Start Vite dev server (http://localhost:5173)
npm run build        # Full build: hashes, proto, type-check, and Vite build
npm run build:proto  # Regenerate protobuf bindings from ../../../../../protobufs
npm run lint         # Run oxlint (Rust-based linter)
npm run type-check   # Run TypeScript compiler without emitting
npm run preview      # Preview production build locally
```

**Testing:** Not configured. No test runner exists in this project.

## Tech Stack

- **React 19** with function components only
- **TypeScript 5.9** with strict mode enabled
- **Vite 7** for bundling (supports top-level await, URDF/STL assets)
- **Tailwind CSS v4** with @tailwindcss/vite plugin
- **Three.js** for 3D rendering (URDF robot visualization)
- **Protobuf.js** for binary protocol communication
- **React Router v7** for routing with lazy-loaded pages
- **oxlint** for linting (fast Rust-based linter)
- **lucide-react** for icons
- **urdf-loader** for loading URDF robot models
- **nosleep.js** for screen wake lock

## Project Structure

```
src/
  api/            # WebSocket, protobuf, time sync, queue, normfs, commands, clipboard, frame parsing
  components/     # Shared UI components
    history/      # History page detail views (ExpandedView, HistoryElement, etc.)
  devices/        # Concrete device modules loaded through bundled dynamic imports
  hooks/          # Custom React hooks (re-exported from index.ts)
  pages/          # Route components (suffixed with Page)
  st3215/         # Motor driver components, utilities, and robot renderers
    robot-rendering/ # Shared ST3215 URDF/Three.js rendering host and base renderer
  usbvideo/       # Camera/video stream components
  utils/          # Shared utilities (asset-hashes, format-bytes, tag-phrases)
public/
  devices/        # Device URDF models and STL assets, keyed by device id
```

## Device Modules

Concrete devices live under `src/devices/<device-id>/`. Put device-specific code there, including:

- React wrappers/views that are only valid for that device
- URDF paths and device asset paths
- joint name mappings
- base position / rotation transforms
- material-to-motor status mapping
- device-specific kinematics, visual behavior, or small business rules

Shared ST3215 code stays under `src/st3215/`. Keep this layer about the bus/protocol and reusable UI/rendering shell:

- motor parsing and ST3215 command helpers
- bus cards, bus viewer, calibration page, mirroring controls
- generic motor tables and camera overlays
- generic robot rendering host/base renderer in `src/st3215/robot-rendering/`

Do not import concrete device modules directly from `src/st3215/` components. Add devices to `src/devices/registry.ts` and load them through bundled dynamic imports:

```typescript
{
  id: 'new-device',
  label: 'New Device',
  protocol: 'st3215',
  matches: (bus) => (bus.motors?.length ?? 0) === 10,
  load: () => import('./new-device'),
}
```

When adding a new physical device:

1. Create `src/devices/<device-id>/index.tsx` and `config.ts`.
2. Put URDF/STL files in `public/devices/<device-id>/`.
3. Register the device in `src/devices/registry.ts`.
4. Run `node scripts/generate-asset-hashes.mjs` from this package so `src/assets-manifest.json` includes the new asset paths.

Avoid hard-coding concrete device names, motor counts, URDF paths, or joint mappings in shared ST3215 components. Use the registry or device module config instead.

## Code Style

### Imports
Use `@/*` path aliases. Order: external deps → `@/api/*` → `@/components/*` → `@/hooks` → types.

Some internal imports use `.js` extensions (required for ESM module resolution):

```typescript
import { forwardRef, memo, useImperativeHandle, useRef } from 'react';
import Long from 'long';
import webSocketManager from '@/api/websocket';
import { serverToLocal } from '@/api/timestamp-utils';
import { st3215 } from '@/api/proto.js';
import Timeline from '@/components/Timeline';
import { useFrameData, useTimelineState } from '@/hooks';
```

### Formatting & Linting
- 2-space indentation, semicolons required
- `src/api/proto.*` files are auto-generated and excluded from linting
- oxlint enforces rules (see `.oxlintrc.json`)

### Naming Conventions

| Entity | Convention | Example |
|--------|------------|---------|
| Components | PascalCase | `TimelineControls`, `BusViewer` |
| Page components | PascalCase + Page suffix | `HomePage`, `HistoryPage` |
| Hooks | camelCase with `use` prefix | `useTimelineState`, `useFrameData` |
| Utilities | kebab-case filenames | `queue-utils.ts`, `time-sync.ts` |
| Variables/functions | camelCase | `currentFrame`, `selectFrame` |
| Constants | UPPER_SNAKE_CASE | `WS_EVENTS`, `DEFAULT_TIMEOUT` |
| Interfaces | PascalCase | `TimelineState`, `ConnectionStats` |
| Props interfaces | PascalCase + Props suffix | `TimelineProps`, `BusViewerProps` |
| Error singletons | Err prefix | `ErrNotConnected`, `ErrBufferFull` |
| Protobuf interfaces | I prefix (from codegen) | `web.IClientPacket`, `st3215.IInferenceState` |

## Component Patterns

### Function Components
Two accepted patterns:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [norma-core/norma-core](https://github.com/norma-core/norma-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
