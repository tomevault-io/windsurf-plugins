---
trigger: always_on
description: **Dove's DataViewer / HackTheTrack** — Open-source, offline-first motorsport telemetry viewer.
---

# CLAUDE.md — Codebase Intelligence for AI Agents

## Project Identity

**Dove's DataViewer / HackTheTrack** — Open-source, offline-first motorsport telemetry viewer.
- Live: [hackthetrack.net](https://hackthetrack.net) | Published: [dovesdataviewer.lovable.app](https://dovesdataviewer.lovable.app)
- Companion hardware: [DovesDataLogger](https://github.com/TheAngryRaven/DovesDataLogger) (ESP32 GPS logger with BLE)
- PWA with full offline support via service worker + IndexedDB

---

## Golden Rules

1. **Offline-first**: 99% of features must work without network. Only weather, satellite tiles, and admin are exceptions.
2. **Modular & reusable**: Prefer small composable modules over monoliths. Rewrites for reusability are always welcome.
3. **Update README.md** when adding parsers, changing env vars, or modifying build params.
4. **Update credits** (in README) when adding new FOSS dependencies.
5. **Never do on the server what you can do on the client.**

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | React 18 + TypeScript |
| Build | Vite + vite-plugin-pwa |
| Styling | Tailwind CSS + shadcn/ui (HSL design tokens in `index.css`) |
| Mapping | Leaflet (CartoDB + Esri tiles, cached 30 days by SW) |
| Charts | Custom Canvas 2D (not a library — see `TelemetryChart.tsx`, `SingleSeriesChart.tsx`) |
| Video Export | WebCodecs + [mp4-muxer](https://github.com/Vanilagy/mp4-muxer) (H.264 video + AAC audio video + AAC audio MP4 output) |
| State | React hooks + React Query (for admin only) |
| Local Storage | IndexedDB (`dbUtils.ts`) for files/metadata/karts/notes/setups/video-sync/graph-prefs; localStorage for tracks & settings |
| Backend | None for core features. Optional admin via Supabase (Lovable Cloud) |
| BLE | Web Bluetooth API for DovesDataLogger device communication |

---

## Architecture Map

```
src/
├── pages/
│   ├── Index.tsx          # Main SPA — file import, tab views, all state orchestration
│   ├── Admin.tsx          # Admin panel (behind VITE_ENABLE_ADMIN)
│   ├── Login.tsx / Register.tsx / Privacy.tsx
│   └── NotFound.tsx
├── components/
│   ├── ui/                # shadcn/ui primitives (button, dialog, tabs, etc.)
│   ├── admin/             # Admin tabs: TracksTab, CoursesTab, SubmissionsTab, BannedIpsTab, ToolsTab, MessagesTab
│   ├── tabs/              # Main view tabs: GraphViewTab, RaceLineTab, LapTimesTab, LabsTab
│   ├── graphview/         # Pro mode: GraphPanel, GraphViewPanel, MiniMap, SingleSeriesChart, InfoBox
│   ├── drawer/            # File manager drawer tabs: FilesTab, KartsTab, NotesTab, SetupsTab, DeviceSettingsTab, DeviceTracksTab
│   ├── track-editor/      # Track editor sub-components
│   ├── RaceLineView.tsx   # Leaflet map with race line, speed heatmap, braking zones
│   ├── TelemetryChart.tsx # Canvas-based speed/telemetry chart (simple mode)
│   ├── VideoPlayer.tsx    # Synced video playback with modular overlay system
│   ├── video-overlays/   # Overlay system for video export
│   │   ├── types.ts             # OverlayInstance, OverlaySettings, DataSourceDef, ThemeDef
│   │   ├── registry.ts          # Overlay type definitions + factory
│   │   ├── themes.ts            # Classic + Neon theme definitions
│   │   ├── dataSourceResolver.ts # Maps data source IDs → values/ranges/units
│   │   ├── DigitalOverlay.tsx   # Numeric value + unit display
│   │   ├── AnalogOverlay.tsx    # Canvas needle gauge (~252° arc)
│   │   ├── GraphOverlay.tsx     # Rolling canvas line chart
│   │   ├── BarOverlay.tsx       # Horizontal 0-100% progress bar
│   │   ├── BubbleOverlay.tsx    # XY joystick-style circular widget
│   │   ├── sectorUtils.ts        # Shared sector status logic (colors, segment computation)
│   │   ├── MapOverlay.tsx       # Mini canvas race line with position dot + optional sector coloring
│   │   ├── PaceOverlay.tsx      # Horizontal pace delta indicator
│   │   ├── SectorOverlay.tsx    # 3 sector bubbles with delta + sparkle animation
│   │   ├── LapTimeOverlay.tsx   # Lap timer with optional pace mode (delta + best lap)
│   │   ├── OverlaySettingsPanel.tsx # Add/configure/remove overlay instances
│   │   └── VideoExportDialog.tsx    # Export dialog with quality options
│   ├── FileImport.tsx     # Drag-and-drop file import
│   ├── DataloggerDownload.tsx  # BLE device download UI
│   ├── ContactDialog.tsx  # Public contact form dialog (categories shared const)
│   └── ...
├── hooks/
│   ├── useSessionData.ts      # Parses imported file → ParsedData
│   ├── useLapManagement.ts    # Lap calculation, selection, visible range
│   ├── usePlayback.ts         # Playback cursor (shared across chart + map)
│   ├── useReferenceLap.ts     # Reference lap overlay logic
│   ├── useVideoSync.ts        # Video ↔ telemetry synchronization
│   ├── useFileManager.ts      # IndexedDB file CRUD
│   ├── useKartManager.ts      # Backward compat re-export → useVehicleManager
│   ├── useVehicleManager.ts   # Vehicle profiles CRUD
│   ├── useTemplateManager.ts  # Vehicle types & setup templates CRUD
│   ├── useNoteManager.ts      # Session notes CRUD
│   ├── useSetupManager.ts     # Generic setup sheets CRUD (template-driven)
│   ├── useSettings.ts         # User preferences (units, smoothing, dark mode, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAngryRaven/DovesDataViewer](https://github.com/TheAngryRaven/DovesDataViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
