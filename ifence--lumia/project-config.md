---
trigger: always_on
description: Lumia is a small, polished, high-performance, cross-platform image viewer for everyday users and professional users such as photographers, UI designers, and engineers. The product should feel fast for ordinary image browsing while still leaving room for professional preview formats and extensibility.
---

# AGENTS.md

## Project Intent

Lumia is a small, polished, high-performance, cross-platform image viewer for everyday users and professional users such as photographers, UI designers, and engineers. The product should feel fast for ordinary image browsing while still leaving room for professional preview formats and extensibility.

The core app must stay small, fast, low-memory, stable, and maintainable. Startup, first image open, folder navigation, zooming, panning, rotation, and metadata display are core quality bars. Heavy decoders, AI, networking, batch processing, and model SDKs must stay outside the core process unless an explicit ADR moves a narrowly scoped capability into core.

Product capability layers:

1. Core viewer: image preview; zoom, pan, and display rotation; image information; EXIF display; folder browsing; basic sorting, filtering, and favorites; fast preview for common formats.
2. Built-in light editing: non-destructive or copy-export operations only, including rotate, crop, mirror, resize, simple compression, simple color adjustments, and export copy.
3. Official bundled plugins: professional and heavier default capabilities such as RAW, HDR, HEIC/HEIF, advanced/professional format preview, and simple format conversion. Users may experience these as default support, but implementation should remain behind the plugin boundary.
4. Optional third-party or advanced plugins: AI stylization, background removal, super-resolution, repair, outpainting, denoising, batch watermarking, batch conversion, compression plugins, cloud model plugins, and local model plugins.

Current transition note: `lumia-core` currently contains HEIC/HEIF decode support. Treat this as a compatibility bridge, not a precedent for adding more heavy decoders to core. Future professional/heavy format work should move toward official bundled plugins.

## Workspace Structure

```
crates/
  lumia-core/src/              -- UI-independent viewer domain
    lib.rs                     -- module declarations and re-exports only
    image.rs + image/          -- facade; types, formats, loading, raster, HEIC bridge
    navigation.rs              -- FolderNavigation scanning and traversal
    viewer.rs                  -- ViewerSession and display-transform state
    viewport.rs                -- ViewportState, FitMode
    settings.rs, task.rs       -- settings and task models

  lumia-plugin-api/src/        -- pure plugin protocol data
  lumia-plugin-host/src/       -- process transport
    lib.rs                     -- declarations and re-exports only
    error.rs, process.rs       -- host errors and stdio JSON-RPC process

  lumia-app/src/               -- GPUI desktop integration
    main.rs, bootstrap.rs      -- CLI/action skeleton and GPUI window startup
    app.rs                     -- LumiaApp state composition and construction
    load_state.rs              -- load generations, queued preloads, decode/cache lifecycle
    image_loading.rs           -- decode, preload, and navigation orchestration
    viewer_actions.rs          -- open, zoom, rotate viewer commands
    window_actions.rs          -- fullscreen, panels, status hover behavior
    preferences.rs             -- settings updates and shortcut bindings
    ui_state.rs                -- pointer, window, menu, overlay, and settings-panel state
    render.rs                  -- root Render implementation and viewer surface
    status_bar.rs              -- status/navigation/zoom controls
    viewer_overlays.rs         -- zoom menu, decode overlay, context menu
    settings_ui.rs             -- settings panel shell and sidebar
    settings_general.rs        -- language/theme settings
    settings_shortcuts.rs      -- shortcut editor
    image_info.rs, widgets.rs  -- image overlay and shared widget factories
    palette.rs, i18n.rs        -- theme palette and translations
    persistence.rs, util.rs    -- settings storage and formatting helpers
    shell.rs + shell/          -- OS dispatch and per-platform registration

plugins/
  lumia-plugin-sample/         -- minimal stdin/stdout JSON-RPC plugin
```
## Crate Dependency Graph

```
lumia-app ──────> lumia-core
    │                  (无工作区依赖)
    └──────> lumia-plugin-host ──> lumia-plugin-api
                                            (无工作区依赖)

lumia-plugin-sample ──> lumia-plugin-api
```

- 无循环依赖
- `lumia-core` 和 `lumia-plugin-api` 是叶子 crate
- `lumia-app` 是唯一的整合点

## Architecture Rules

- Use Rust and GPUI for the desktop application.
- Keep UI code in `crates/lumia-app` thin; put reusable viewer state and task models in `crates/lumia-core`.
- Keep the core viewer path optimized for startup time, open latency, memory use, and crash isolation.
- Do not add heavy decoder, AI, networking, batch-processing, or model SDK dependencies to `lumia-app`.
- Do not add new heavy/professional format decoders to `lumia-core`; route them through official bundled plugins unless an ADR explicitly approves a core exception.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iFence/Lumia](https://github.com/iFence/Lumia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
