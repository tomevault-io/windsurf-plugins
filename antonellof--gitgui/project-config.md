---
trigger: always_on
description: A pixel-rendered git GUI that runs inside kitty-graphics terminals (Ghostty, cmux, kitty). No Chromium, no Electron. A single Rust binary renders an egui UI into an RGBA framebuffer, ships frames to the terminal with the kitty graphics protocol, and reads pixel-precise mouse and keyboard input back from the terminal.
---

# gitgui

A pixel-rendered git GUI that runs inside kitty-graphics terminals (Ghostty, cmux, kitty). No Chromium, no Electron. A single Rust binary renders an egui UI into an RGBA framebuffer, ships frames to the terminal with the kitty graphics protocol, and reads pixel-precise mouse and keyboard input back from the terminal.

Reference projects for the idea (not the implementation): zenbu-labs/terminal-browser and zenbu-labs/terminal-code. We reuse their trick (pixels in the terminal via kitty graphics + synthetic input) but skip the browser engine entirely.

Read `docs/SPEC.md` (architecture, UI, git layer, milestones) and `docs/PROTOCOLS.md` (exact escape sequences) before writing code. They are the source of truth. If the spec and this file disagree, the spec wins.

## Stack

- Rust, latest stable, edition 2021
- `egui` + `epaint` for the UI model and tessellation (we do NOT use eframe or any GPU backend)
- Our own software rasterizer for egui meshes (`src/render/raster.rs`)
- `git2` (libgit2) for all repository reads and index/commit writes; `git` CLI subprocess only for network ops (fetch, pull, push)
- `libc` for termios, ioctl, POSIX shared memory
- `flate2` + `base64` for the SSH fallback transport
- `png` crate for headless frame dumps used in tests

No async runtime. One thread for the UI loop, one thread that reads stdin into a channel, one worker thread for slow git operations.

## Module map

```
src/
  main.rs            wires modules, mode dispatch (interactive, headless-frame, dump-input, probe)
  cli.rs             argument parsing and --help
  runtime.rs         interactive main loop: input channel, egui run, raster, frame send, resize
  term/
    mod.rs           raw mode, alt screen, enable/disable sequences, restore on exit and panic
    probe.rs         capability probing: kitty graphics, kitty keyboard, cell size, pixel size
    input.rs         byte stream -> Event (keys, mouse in pixels, resize, focus, paste)
    kitty.rs         kitty graphics encoder: shm transport, direct transport, place, delete
  render/
    raster.rs        triangle rasterizer for epaint meshes, texture atlas, clip rects
    frame.rs         double-buffered RGBA framebuffer, dirty detection, headless PNG export
  git/
    repo.rs          Repository wrapper: status, branches, log, diffs, stage/unstage, commit
    graph.rs         commit graph lane assignment
    ops.rs           slow ops on the worker thread (fetch/pull/push via git CLI)
  ui/
    app.rs           top-level egui app state, panels, footer, modals, keybindings
    sidebar.rs       branches, remotes, tags, stashes
    log.rs           commit list with graph column
    changes.rs       working tree: unstaged/staged file lists, commit box, layout math
    diff.rs          diff viewer with per-hunk stage/unstage, wrap toggle
    toolbar.rs       footer buttons (fetch, pull, push, refresh, quit), icon-only below 560 pt
    branch_picker.rs branch switcher modal
    row.rs           row helper: trailing widgets from the right, leading side clipped
    input.rs         egui RawInput from terminal events
    icons.rs, logo.rs small painted glyphs
    theme.rs         colors derived from terminal palette (OSC 10/11 query, fallback dark)
  split.rs           open in a terminal split (cmux, Ghostty) with in-place fallback
  agent.rs           unix socket JSON-lines control API (phase 5)
```

## Pinned versions

Toolchain at kickoff: rustc 1.98.0, cargo 1.98.0 (2026-08). Both crates below are pinned with `=` in Cargo.toml.

- `egui = "=0.36.1"`, `epaint = "=0.36.1"` (rust-version 1.95). API notes:
  - `egui::Context::run` no longer exists. Use `ctx.run_ui(raw_input, |ui| ...)` which returns `FullOutput`, or `begin_pass` / `end_pass`.
  - `FullOutput` fields: `platform_output`, `textures_delta`, `shapes: Vec<ClippedShape>`, `pixels_per_point`, `viewport_output`.
  - `ctx.tessellate(shapes, pixels_per_point) -> Vec<ClippedPrimitive>`.
  - `epaint::ImageData` has a single variant `Color(Arc<ColorImage>)`. There is no `Font` variant; the font atlas arrives as premultiplied RGBA `ColorImage` (`pixels: Vec<Color32>`, `as_raw()` for bytes). `ImageDelta { image, options, pos: Option<[usize; 2]> }`.
  - `TexturesDelta` lives at `epaint::textures::TexturesDelta`. `set` is `HashMap<TextureId, SmallVec<[ImageDelta; 1]>>` (apply each delta in order), `free` is `HashSet<TextureId>`.
  - Dropping a `TexturesDelta` without applying it panics in debug builds, even after you applied it by reference: call `clear()` once done.
  - Panels: `SidePanel` and `TopBottomPanel` are gone. Use `egui::Panel::left("id").default_size(220.0).show(ui, ...)`, `Panel::bottom(...)`, and `CentralPanel::default().show(ui, ...)`. All take the root `&mut Ui` that `run_ui` hands to the closure, not a `Context`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonellof/gitgui](https://github.com/antonellof/gitgui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
