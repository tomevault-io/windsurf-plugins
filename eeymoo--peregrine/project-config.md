---
trigger: always_on
description: This file is intended for AI coding agents working in this repository. It helps you quickly understand the Peregrine project and make changes safely, even without prior context. The content is verified against the actual codebase; if anything conflicts with the code, the code takes precedence.
---

# AGENTS.md

This file is intended for AI coding agents working in this repository. It helps you quickly understand the Peregrine project and make changes safely, even without prior context. The content is verified against the actual codebase; if anything conflicts with the code, the code takes precedence.

## Project Overview

Peregrine is a desktop visual anchor (overlay) tool. **Its primary purpose is to reduce 3D motion sickness**: it draws semi-transparent visual anchors at the center or edges of the screen, giving players a fixed reference point in 3D games to alleviate dizziness.

- Language / ecosystem: **Rust**, Cargo **workspace** (`resolver = "3"`, `edition = "2024"`, `rust-version = 1.85`, MIT licensed).
- Graphics stack: **Tauri** (settings window Webview) + **React + Tailwind + shadcn/ui** (settings panel). The overlay still uses `winit` + `softbuffer`; the original `wgpu` + `egui` implementation has been removed.
- Async runtime: `tokio` (configuration read/write, file hot-reload, background follow task).
- Target platform: **Windows** (x86 / x86_64 / ARM64). Overlay transparency / click-through / window-following capabilities are intentionally Windows-only and are not planned for other platforms.
- Current status: **v0.2.1 stable released**. Windows transparent, always-on-top, click-through overlay; target window following; 12 crosshair styles; custom PNG decals; multi-profile config; configuration hot-reload; in-app auto-updater (NSIS); GlitchTip telemetry; bilingual UI (zh-CN / en) are all functional. "Process trigger" remains a configuration placeholder.
- **Static layer rendering + dynamic material pipeline both ENABLED** (change `restore-dynamic-material` supersedes the earlier soft-disable era of `disable-material-runtime` / `material-static-rendering`): multi-layer rendering via layers + Rhai materials is active (`MATERIAL_RUNTIME_ENABLED = true`) — overlay and preview render `Profile.layers` through `build_layers_shapes`, WYSIWYG. The **dynamic** pipeline is also restored (`MATERIAL_DYNAMIC_INPUT_ENABLED = true`, declared in both `crates/peregrine/src/lib.rs` and `src/lib/feature.ts`) under a **two-layer AND gate**: dynamic links (input polling, dynamic eval context, continuous redraw, picker visibility) are live only when the compile-time constant AND the runtime user switch `settings.material.dynamic_enabled` (default `true`, Settings → Materials tab) are both on. `builtin.time` (clock) is a built-in dynamic material using the context snapshot `time_ms()` (not wall-clock `now_ms()`). Continuous redraw cadence derives from `settings.material.fps` (30/60/120, `None` = follow monitor refresh rate, fallback 60) and hot-updates on `UpdateConfig`; purely static profiles stay event-driven (`ControlFlow::Wait`). The animacy check is recomputed every `about_to_wait` (no cache field); `OverlayCommand::RefreshMaterials` carries a fresh `Arc<MaterialRegistry>` and the material watcher also emits `peregrine:materials-changed` to the frontend; the preview re-polls `build_shapes_ipc` at ~1s when the profile contains a dynamic material. To soft-disable dynamic input again, flip both `MATERIAL_DYNAMIC_INPUT_ENABLED` constants to `false` (runtime switch + FPS setting become unconsumed UI fields, harmless); to fully soft-disable the material runtime, flip `MATERIAL_RUNTIME_ENABLED` to `false`. Gated sites are greppable via the constant names.

All code comments and documentation use **Simplified Chinese**. Please continue writing new comments, documentation, and commit message bodies in Chinese for consistency.

## Repository Structure

```
peregrine/
├── Cargo.toml            # workspace root: members, workspace.package, workspace.dependencies, build profiles
├── Cargo.lock
├── .gitignore            # ignores /target, *.log, .DS_Store, docs build artifacts, etc.
├── assets/               # application icon (icon.ico) and icon generation script (gen_icon.py)
├── docs/                 # VitePress documentation site (deployed to GitHub Pages)
│   ├── .vitepress/       # VitePress config (config.mts), theme, dist/ build output
│   ├── guide/            # user guide, introduction, quick start, features, configuration, development/build
│   ├── public/           # static assets (logo.svg, etc.)
│   ├── index.md          # documentation homepage
│   └── package.json      # vitepress + mermaid + llms plugins
├── .github/workflows/    # ci.yml (three-platform compile + lint), release.yml (tag-based release), snapshot.yml (unsigned test builds), pages.yml (docs deployment)
├── .agents/skills/       # AI agent skill definitions (unified; opencode + Agent Skills spec)
├── src-tauri/            # peregrine-tauri: Tauri backend entry, tray, commands, overlay management
│   ├── Cargo.toml
│   ├── build.rs
│   ├── tauri.conf.json
│   ├── capabilities/
│   ├── icons/
│   └── src/
│       ├── lib.rs             # Tauri startup entry: config init, tray, commands, watcher
│       ├── main.rs            # binary entrypoint, calls lib::run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eeymoo/peregrine](https://github.com/Eeymoo/peregrine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
