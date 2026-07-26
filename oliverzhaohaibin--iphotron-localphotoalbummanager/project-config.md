---
trigger: always_on
description: This file is the working guide for coding agents and contributors. It reflects
---

# AGENT.md - iPhotron Development Principles

This file is the working guide for coding agents and contributors. It reflects
the current vNext state: the production runtime has converged on
`RuntimeContext -> LibrarySession -> application ports/services`, and legacy
compatibility code is quarantined under `src/iPhoto/legacy/`.

## 1. Current Architecture Status

- The vNext cleanup is complete for production source code.
- Production runtime code must not import `iPhoto.legacy` or `iPhoto.models.*`.
- Legacy compatibility and old domain-repository code live only in
  `src/iPhoto/legacy/`. That subtree is temporary quarantine for historical
  behavior tests and is planned for removal in the next major release.
- GUI, CLI, file watchers, Qt workers, and future automation entry points must
  enter library behavior through `RuntimeContext`, `LibrarySession`, and
  application-level surfaces.
- New business logic belongs in application use cases/services, session
  surfaces, domain values/pure services, or infrastructure adapters. GUI code
  is presentation and Qt transport only.

The authoritative current architecture is tracked in `docs/architecture.md`.
Completed vNext migration records are archived under
`docs/finished/refactor/vnext-2026-06/`.

## 2. Product Invariants

- **Folder-native library.** A folder is an album. Users can browse folders
  without an import step.
- **Local-first.** Core library, browsing, editing, Live Photo, People, and Maps
  behavior is local. Optional runtimes must degrade gracefully when unavailable.
- **Non-destructive editing.** Visual edits are stored in `.ipo` sidecars.
  Original media is not overwritten by normal editing.
- **Explicit metadata write-back only.** Assign Location is the explicit
  exception: it persists the location locally first, then best-effort writes GPS
  metadata to the original file through ExifTool and reports warnings on
  failure.
- **Rebuildable facts vs durable choices.** Scan facts, thumbnails, Live Photo
  materialization, and People runtime snapshots can be rebuilt. Favorites,
  hidden/trash state, pinned items, album order, manual metadata, People names,
  covers, groups, group order, hidden flags, and manual faces must survive
  rescans and rebuilds.
- **Cross-platform desktop first.** macOS, Windows, and Linux remain supported.
  Platform-specific rendering, maps, ExifTool, FFmpeg, and AI behavior must be
  isolated behind adapters or runtime discovery.

## 3. Runtime And Layering Rules

The production dependency direction is:

```text
gui -> bootstrap/runtime -> application -> domain
infrastructure -> application ports / domain values
bounded contexts -> application ports / domain values
```

Forbidden directions:

```text
domain -> application/gui/infrastructure
application -> gui/concrete cache/concrete infrastructure
infrastructure/cache/core/io/library/people -> gui
production runtime -> iPhoto.legacy
production runtime -> iPhoto.models.*
```

Key runtime objects:

- `RuntimeContext`: process composition root, current settings/theme/recent
  libraries, active `LibrarySession` lifecycle.
- `LibrarySession`: library-scoped adapters and surfaces for assets, state,
  scanning, album metadata, People, Maps, thumbnails, edit sidecars, location,
  asset lifecycle, and file operations.
- `LibraryRuntimeController`: GUI/runtime controller bound to the active
  session; it should not re-create standalone compatibility services.

Compatibility code is not a production extension point. Do not add new features
to `src/iPhoto/legacy/app.py`, `src/iPhoto/legacy/appctx.py`,
`src/iPhoto/legacy/bootstrap/*`, or other quarantine modules.

## 4. Files And State

Album markers:

- `.iphoto.album.json`: folder-local album manifest.
- `.iphoto.album`: minimal marker for folder-native album discovery.
- `.iPhoto/manifest.json`: compatibility manifest location supported by the
  current manifest repository.

Library workspace:

```text
/<LibraryRoot>/.iPhoto/
  global_index.db       # SQLite index and current asset/state repository store
  links.json            # Live Photo compatibility materialization
  cache/thumbs/         # Rebuildable thumbnail cache
  faces/
    face_index.db       # Rebuildable People runtime snapshot
    face_state.db       # Durable People user decisions
    thumbnails/         # Rebuildable cropped face thumbnails
  manifest.bak/         # Manifest/links backup area
  locks/                # File-level locks for JSON sidecars
```

State rules:

- `global_index.db` is the current source of truth for asset scan rows,
  pagination, Live Photo roles, trash/favorite/hidden flags, face scan status,
  and the repository-backed user-state boundary.
- Large-library gallery reads are SQL-first and windowed through collection
  query APIs. Normal visible rows must be thumbnail-ready and carry a
  `thumb_cache_key`.
- Gallery paint/model access is memory-only. Sparse rows load asynchronously;
  viewport generations reject stale window and thumbnail results, and visible,
  guard, and far-speculative thumbnail lanes keep separate capacity.
- `links.json` is derived compatibility materialization for Live Photo payloads;
  target runtime behavior should read roles through repository/session surfaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OliverZhaohaibin/iPhotron-LocalPhotoAlbumManager](https://github.com/OliverZhaohaibin/iPhotron-LocalPhotoAlbumManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
