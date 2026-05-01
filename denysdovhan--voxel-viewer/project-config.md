---
trigger: always_on
description: This project is a static, local-first CBCT viewer and installable PWA.
---

# Agents Rules

## Project Workflow

This project is a static, local-first CBCT viewer and installable PWA.

Current runtime flow:

1. `src/App.tsx` mounts `BrowserRouter` with basename derived from the active Vite base path.
2. `src/app/AppRouter.tsx` creates the scan-folder picker, lazy-loads pages, and redirects between `/` and `/viewer` from app state.
3. `src/lib/import/source-picker/*` prefers `showDirectoryPicker`, then falls back to `webkitdirectory` upload when available.
4. `src/app/useViewerApp.ts` owns import lifecycle, progress/errors, window/level draft state, MPR zoom, selected axis, and sidebar visibility.
5. `src/lib/import/adapters/*` matches the selected folder layout and parses format-specific metadata.
6. `src/lib/import/load-volume.ts` parses metadata on the main thread, then posts one stable request to `src/workers/volume.worker.ts`.
7. `src/workers/volume/assemble/*` decodes voxels by format and returns histogram / scalar metadata.
8. `src/lib/volume/preview-3d.ts` prepares the 3D texture payload, including quantization, threshold estimation, and GPU-safe downsampling.
9. `src/pages/ViewerPage.tsx` renders the responsive viewer shell with `VolumeViewport3D`, `AxisViewportGrid`, and `ViewerSidebar`.
10. `src/sw.ts` caches the built app shell for offline/PWA use; scan folders are still reopened each session.

## Working Areas

```text
src/
├── App.tsx
├── app/
│   ├── AppRouter.tsx
│   ├── useViewerApp.ts          # main state owner; avoid pushing readiness/UI decisions down into loaders
│   └── viewer-layout.ts         # controls the compact viewer breakpoint: `max-width: 767px`
├── components/
│   ├── AxisViewportGrid.tsx
│   ├── SliceCanvas.tsx          # owns MPR interaction behavior: scrub, wheel zoom, pinch zoom
│   ├── ViewerSidebar.tsx        # owns window/level sliders and sidebar visibility behavior
│   └── VolumeViewport3D.tsx     # owns Three.js mount/retry behavior and viewport overlay controls
├── lib/
│   ├── import/                  # import orchestration logic
│   │   ├── adapters/            # format-specific data parsing; keep grouped by format
│   │   ├── source-picker/       # source picking capability fallback; keep `showDirectoryPicker` first
│   │   └── load-volume.ts       # orchestration-only; keep parsing and decoding logic out of the main thread
│   └── volume/                  # volume data handling and 3D preview prep
│       ├── three-preview/       # Three.js preparation and rendering logic
│       └── preview-3d.ts        # 3D preview logic
├── pages/                       # top-level route components
│   ├── ImportPage.tsx           # route for homepage and scan folder picking
│   └── ViewerPage.tsx           # route for the viewer
├── sw.ts                        # service worker entrypoint
└── workers/
    ├── volume.worker.ts         # thin worker entrypoint
    └── volume/                  # format-specific voxel decoding; keep all heavy lifting off the main thread
        ├── assemble/            # format-specific assembly logic
        └── scalars.ts           # shared scalar metadata extraction logic
```

## Data References

```text
ct/
├── galileos/
└── onevolume/
    ├── CT_20250225114353/
    ├── DICOM/
    └── Series_01/
```

- inspect `ct/` before changing import / reconstruction logic
- use `ct/onevolume/CT_20250225114353/` for native OneVolume validation
- use `ct/onevolume/DICOM/` for DICOM validation
- `ct/onevolume/Series_01/` is reference-only; do not treat it as an import source
- older notes mentioning `xray/`, `xray-onevolume/`, or `original-software/` are stale

## Technical Decisions

- keep routing and assets base-path-safe; router basename, service worker registration, manifest links, and GitHub Pages deploy all depend on `import.meta.env.BASE_URL` / `VITE_BASE_PATH`
- keep heavy voxel decode and 3D prep off the main thread
- keep one worker entrypoint, with format-specific assembly under `src/workers/volume/assemble/*`
- keep import parsing grouped by format under `src/lib/import/adapters/<format>/`
- support three import formats:
  - GALILEOS folder with exactly one `*_vol_0` header and contiguous `*_vol_0_###` slices
  - OneVolume export with exactly one `CT_0.vol`
  - DICOM slice folder with at least two `.dcm` files
- DICOM detection currently relies on `.dcm` extensions; if broadening support, update matcher, parser, and homepage copy together
- keep directory-handle picking as the first choice, but preserve the upload fallback for browsers without `showDirectoryPicker`; the current mobile fallback matters for Safari / iOS
- the upload fallback is gated for iOS Safari; current hint text assumes `webkitdirectory` support on iOS 18.4+
- keep `volume` as the source of truth for whether the app is in import mode or viewer mode; router syncing is secondary
- keep coronal and sagittal views superior-at-top
- keep window/level sliders on draft state plus debounced commit (`96ms`)
- compact viewer mode is not cosmetic: it switches to one MPR pane at a time and turns the sidebar into an overlay drawer
- 3D is the primary large viewport; preserve the current Three.js renderer path unless the user explicitly asks to replace it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysdovhan/voxel-viewer](https://github.com/denysdovhan/voxel-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
