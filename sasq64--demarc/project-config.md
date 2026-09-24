---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`demarc` is a command-line emulator frontend for watching demoscene productions: it takes files,
or a demo database, figures out which machine each one belongs to, downloads/loads the right
libretro core, and plays them full screen through a CRT/LCD shader — optionally several at once in
a grid. Single Rust binary, Bevy 0.19 for the app/render loop, edition 2024.

## Important Notes

- Avoid long comments, and comments that describe current behaviour, even though there are many comments like that in the code.
- When given a coding task, prefer simple small changes. Fix problems if needed, but don't embellish (don't add information to the UI unless told).

## Commands

Standard cargo commands

Good profile for testing:
```sh
cargo build --profile release-fast    
```

## Testing changes

Run headless with

`<files> --headless --remote-control <lua_script>`

See `scripts/remote_example.lua`

*NOTE* Avoid running normally since it opens windows and plays audio.


## Architecture

### Layers

```
main.rs            CLI (clap, src/config.rs) → Bevy App + plugins;
  frontend.rs      FrontendPlugin: spawns one Emulator entity per view, grid layout, run_retro main system
    emulator.rs    Emulator component: pacing, input routing, audio sink, frame → Handle<Image> upload
      backend.rs   `trait Backend` — the only thing the frontend knows about a "core"
  post_process.rs  librashader/wgpu compositing of every view into one camera
  egui_ui.rs       HUD, info overlay, fuzzy-search selector (fuzzy_list.rs)
  commands.rs      RightAlt/RightCtrl hotkeys → Cmd enum → app actions
```

`Backend` implementations, all interchangeable from the frontend's point of view:

| impl | file | notes |
|---|---|---|
| libretro core | `retro_emu.rs` + `retro_emu/threaded.rs` | `RetroCoreDirect` is the raw FFI/environment-callback side; `RetroCoreThreaded` runs it on a worker thread and ships frames back over a channel. This is what almost everything uses. |
| still image | `image_emu.rs` | IFF/ILBM (`ilbm.rs`), DEGAS (`degas.rs`), ZX SCR (`zx_scr.rs`), palette TIFF (`tiff_pal.rs`, which the `image` crate refuses), plus `image` crate formats; optional palette colour-cycling |
| music | `music_emu.rs` | `musix` chiptune/tracker player, renders audio inline (no worker thread) and draws a Luau visualizer (`music_vis.rs`) |
| Flash | `flash_emu.rs` | behind the `flash` feature; Ruffle with its own wgpu device |
| gamescope session | `external/gamescope/src/libretro/` (C++) | Linux only. A patched gamescope composites a headless Wayland/Xwayland session into a shared dmabuf and a thin `gamescope_libretro.so` hands the frames back, so wine — or an HTML/JS release in an undecorated Chrome — is a picture source like any other. What to run inside it is `src/wine.rs`'s job for a Windows release (the wine command, the prefix, its teardown), restated as core options by `src/newsys/windows.rs`. See `docs/GAMESCOPE.md`. |

### System detection — `newsys.rs` + `src/newsys/*`

`trait System` is the per-machine knowledge: which extensions/headers it claims (`can_load`),
what to do to a release before it can boot (`load`, working on a mutable `WorkFile`), which libretro
core to use (`core_name`), default core options (`default_meta`), and how to build the backend
(`create`, whose default just spins up `RetroCoreThreaded`).

`NewSys::load_file` is the whole pipeline: unpack archives (twice, releases are often double-packed) →
read m3u tags → apply `overrides.toml` → apply `-x` CLI meta → try each `System` in the order listed
in `NewSys::get_systems` until one claims the file → fill in that system's `default_meta` for keys
nothing else set → `create` the backend. **Order matters** — specific systems come first,
`MusicSystem` and `ImageSystem` last because `musix` and `image` claim a lot of files.

It is split in two so the frontend can run the halves in different places: `newsys::unpack_release`
(unpack + m3u tags) needs nothing but the file, so `Emulator::load_async` runs it on the I/O pool
along with the download, and only `NewSys::load_prepared` (everything from the override onwards)
runs on the main thread — unpacking there cost a dropped frame.
`load_file` is still the two called in order, and is what the tests use.

To add a machine: new file under `src/newsys/`, implement `System`, register it in `get_systems()`.

### Meta

Everything system-specific travels as string key/value "meta" on the `WorkFile`, and keys that a
libretro core announced via `SET_VARIABLES` become that core's options (a value we already hold beats
the core's announced default). Precedence, weakest first: core default → `System::default_meta` →
db line / m3u tags → `overrides.toml` → `-x key=value` on the command line.

`src/overrides.rs` holds per-release fixups keyed on demozoo id (which file to fetch, which file to
boot, files to patch in, AmigaDOS assigns, core options), read from `system/overrides.toml`.

### Files, downloads, cores

- `files.rs` — collects `EmuFile`s from paths, directories and tab-separated demo databases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sasq64/demarc](https://github.com/sasq64/demarc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
