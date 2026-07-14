---
trigger: always_on
description: BoardRipper — web-based PCB boardview file viewer and inspector. Hosted via Docker on NAS.
---

# BoardRipper — Project Configuration

## Project Overview
BoardRipper — web-based PCB boardview file viewer and inspector. Hosted via Docker on NAS.

## License
AGPL-3.0. See [LICENSE](LICENSE) and [THIRD_PARTY.md](THIRD_PARTY.md). AGPL was
chosen because the Allegro parser (`src/frontend/src/parsers/allegro/`) is a
TypeScript re-implementation derived from KiCad (GPL-3.0), which forces the
whole project to be GPL-3.0-compatible. AGPL additionally closes the SaaS
loophole. All other parsers (BVR/BRD/BDV/FZ/CAD/XZZ) draw from OpenBoardView
(MIT); TVW draws from eagleview (MIT). All runtime dependencies are
MIT/Apache-2.0/BSD.

## Tech Stack
- **Rendering:** PixiJS v8 (WebGL) + pixi-viewport v6 (pan/zoom/culling/deceleration)
- **Frontend:** React 19 + TypeScript + Vite 7
- **Panels:** Dockview v5 (dockable, detachable, floating, popout-to-window)
- **Backend:** Go (net/http stdlib) — serves SPA, file management, board database, self-update
- **Container:** Docker (multi-stage build, scratch-based, ~15MB)
- **Tests:** Playwright (Chromium headless)

## Supported Formats
- **BVR1** — tab-delimited, absolute coords ×1000. Spec: `docs/formats/BVR_FORMAT.md`
- **BVR3** — keyword-value, relative pin coords. Spec: `docs/formats/BVR_FORMAT.md`
- **BRD** — binary obfuscated boardview (Apple/Mac repair). Spec: `docs/formats/BRD_FORMAT.md`
- **BDV** — plain-text boardview (BRDOUT/NETS/PARTS/PINS/NAILS sections). Spec: `docs/formats/BDV_FORMAT.md`
- **BDV ASC** — Honhan / Tebo-ICT obfuscated multi-section ASC (line-key cipher). Spec: `docs/formats/BDV_ASC_FORMAT.md`
- **FZ** — ASUS boardview (RC6-encrypted, zlib-compressed). Spec: `docs/formats/FZ_FORMAT.md`
- **CAD** — GenCAD 1.4 text-based PCB interchange. Spec: `docs/formats/CAD_FORMAT.md`
- **XZZ** — XZZ PCB (DES-encrypted boardview). Spec: `docs/formats/XZZ_FORMAT.md`
- **TVW** — Teboview binary (multi-layer, traces, drill data). Spec: `docs/formats/TVW_FORMAT.md`
- **MENTOR_NEUTRAL** — Mentor Graphics Boardstation/Expedition neutral export (text, `.cad` extension; not GenCAD). Spec: `docs/formats/MENTOR_NEUTRAL_FORMAT.md`
- **ALLEGRO_BRD** — Cadence Allegro binary PCB. Two parser families share `parsers/allegro/`:
  - v16.x / v17.x / v18.x (magic `0x0013xxxx` / `0x0014xxxx` / `0x0015xxxx`) — original target. Spec: `docs/formats/ALLEGRO_BRD_FORMAT.md`
  - v15.x (magic `0x0012xxxx`) — added in v0.17.0 via blind RE; ~99% net coverage on 15.5.7 corpus, partial on 15.5.2. Spec: `docs/formats/ALLEGRO_V15_FORMAT.md`

## Project Structure
```
Boardviewer/
├── CLAUDE.md                    # This file
├── README.md
├── Dockerfile                   # Multi-stage build (node → golang → scratch)
├── docker-compose.yml
├── desktop/                     # Electron desktop app (Mac + Windows builds)
├── scripts/                     # CI/workflow scripts
├── Board Database/              # Reference board database (SQLite)
├── docs/
│   ├── formats/                  # Format specifications (one per format)
│   │   ├── BVR_FORMAT.md         # BVR1/BVR3
│   │   ├── BRD_FORMAT.md         # BRD (Apple/Mac obfuscated)
│   │   ├── BDV_FORMAT.md         # BDV (plain-text boardview)
│   │   ├── BDV_ASC_FORMAT.md     # BDV ASC (Honhan / Tebo-ICT obfuscated)
│   │   ├── FZ_FORMAT.md          # FZ (ASUS RC6-encrypted)
│   │   ├── CAD_FORMAT.md         # GenCAD 1.4
│   │   ├── XZZ_FORMAT.md         # XZZ PCB (DES-encrypted)
│   │   ├── TVW_FORMAT.md         # Teboview (multi-layer binary)
│   │   ├── MENTOR_NEUTRAL_FORMAT.md # Mentor Boardstation Neutral (.cad text)
│   │   ├── ALLEGRO_BRD_FORMAT.md # Cadence Allegro v16/v17 BRD
│   │   └── ALLEGRO_V15_FORMAT.md # Cadence Allegro v15.x BRD (RE'd in v0.17.0)
│   ├── PDF_VIEWER.md             # PDF render-pipeline architecture
│   └── RELEASE_RUNBOOK.md        # Maintainer release-cutting procedure
├── samples/                     # Local-only board fixtures (not redistributed)
├── landing/                     # Static landing page deployed to ripperdoc.de
├── scripts/                     # release.sh, packaging, NAS deploy helpers
└── src/
    ├── frontend/                # React + PixiJS SPA
    │   ├── tests/               # Playwright E2E specs
    │   └── src/
    │       ├── parsers/         # Format parsers (pure TS functions, 11 formats)
    │       │   └── allegro/     # Allegro v15.x + v16/v17 (split families share types)
    │       ├── renderer/        # BoardRenderer, board-scene (shared), mockup-data
    │       ├── pdf/             # PDF glyph extraction & overlay utilities
    │       ├── components/      # Toolbar, StatusBar, TabBar, ContextMenu, PanelAdder, BindLink, BoardSidebar, UpdateProgressOverlay, overlay/ (board overlay slots), …
    │       ├── panels/          # BoardViewer, ComponentInfo, NetList, SearchResults, PDF, Settings, SettingsMockup, Debug, Library
    │       ├── hooks/           # useBoardStore, usePdfStore, useDatabank, useObdForBoard, useKeyboardShortcuts, createStoreHook
    │       └── store/           # board-store, render-settings, board-cache, pdf-store, databank-store, update-store, theme-store, overlay-store, obd-store, …
    └── backend/                 # Go net/http server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexeyInwerp/BoardRipper](https://github.com/AlexeyInwerp/BoardRipper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
