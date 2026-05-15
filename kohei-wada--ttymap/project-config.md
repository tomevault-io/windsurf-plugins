---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ttymap is a terminal-based map viewer written in Rust. It renders Mapbox Vector Tiles (MVT/protobuf) as Unicode Braille characters in the terminal, similar to mapscii. Default tile source is `http://mapscii.me/`.

## Build & Development

```bash
cargo build              # build (runs build.rs to compile proto/vector_tile.proto via protox)
cargo run                # run with defaults (Berlin, auto-zoom)
cargo run -- --lat 35.68 --lon 139.76 --zoom 10  # custom location
cargo run -- --style bright                        # alternate style
cargo test               # run all tests
cargo test test_name     # run a single test
cargo clippy             # lint
```

The build step compiles `ttymap-engine/proto/vector_tile.proto` using protox (no system protoc required). The generated Rust code is included at runtime via `include!(concat!(env!("OUT_DIR"), "/vector_tile.rs"))` in `ttymap-engine/src/map/tile/decode/mod.rs`.

## Workspace layout

The repository is a two-crate Cargo workspace:

- `ttymap-engine/` (`ttymap-engine`) — headless rendering engine. Owns the
  map subsystem (tile fetch + decode + cache, render thread, styler),
  the `MapFrame` produced for display, the colour-palette data, the
  `geo` projection module, and the User-Agent-tagged HTTP client. **No
  ratatui or crossterm dependency** — the engine is consumable on its
  own (e.g. by the `snap` subcommand) without bringing in a TUI
  framework. Communication out is a `FrameSink` callback (`Box<dyn
  FnMut(MapFrame) -> bool + Send>`) so the engine never names the
  binary's `AppEvent` bus.
- `ttymap-tui/` (`ttymap-tui`) — TUI binary. Owns the App event loop,
  ratatui draw entry, compositor / palette / input / lua bridge, plus
  the ratatui-side theme adapter (`UiTheme`, `StyleKind`). Wraps
  `ttymap_engine::Config` with binary-only knobs (`geoip`, `runtime`,
  `plugins`, keybinding overrides) — engine-side fields are reached
  via `config.engine.<sub>.<field>`. The crate is named `ttymap-tui`
  but the produced executable is still `ttymap` (set via
  `[[bin]] name = "ttymap"`), so `cargo install` and
  `~/.cargo/bin/ttymap` are unchanged.

## Design philosophy

See [docs/design.md](docs/design.md) for load-bearing design decisions:
- **When to emit a `UserCommand` vs a direct method call** — user intent goes through `App::dispatch`; internal data flow (frame arrival, widget polling) does not.
- **Controller split: by feature, not by domain** — if `App::dispatch` + cross-cutting helpers grow large.
- **Cleanup via `Drop`, not manual** — `RenderHandle`'s thread shutdown is handled by its Drop impl.
- **Frames are completed products** — main thread displays, does not compute.

For the full system architecture (src tree, layering, message + render flow, focus model, concurrency) see [docs/architecture.md](docs/architecture.md). The summary below is enough to navigate the code; details belong in that doc.

## Architecture

### Source tree

The binary is **flat by feature**, Neovim-inspired. We tried a strict
`core/front` split (issue #212 Phase 4) and reverted it — it forced
too many exceptional placements (sidebar policy is "UI" but lived in
core because dispatcher owned it; theme_id leaked into core because
every command tracked it; etc.). The engine/binary boundary is a
genuine layering boundary so it lives at the **crate** level instead.

```
ttymap-engine/                (ttymap-engine — ratatui-free)
  src/
    config.rs                 engine-side settings (cache / map / render)
    geo.rs                    Web Mercator projection math
    map/                      tile + render + styler + viewport state
    shared/http/              User-Agent-tagged reqwest wrapper
    theme/                    palette data (ColorPalette + DARK/BRIGHT + ThemeId)
  proto/vector_tile.proto
  build.rs                    compiles MVT proto via protox
  benches/                    decode_tile / render_frame / tile_disk_hit

ttymap-tui/                   (ttymap-tui — ratatui + crossterm shell)
  src/
    command.rs                UserCommand vocabulary
    config.rs                 wraps ttymap_engine::Config (+ geoip/runtime/plugins)
    logging.rs                XDG state log
    app/                      event loop + dispatcher + ratatui draw entry
      mod.rs / dispatcher.rs / event.rs / frame_timer.rs / frame_widget.rs / ui.rs
    cli/                      CLI subcommands (snap)
    compositor/               focus stack + Component trait + Op + render
    input/                    keymap + mouse adapter + input thread
    palette/                  `:`-triggered picker UI
    theme/                    ratatui adapter — UiTheme + StyleKind
                              (re-exports ColorPalette/ThemeId/DARK/BRIGHT
                               from the engine)
    lua/                      plugin runtime — bridges binary (Component,
                              palette) and engine (MapApi, http)
    shared/geoip.rs           IP → lon/lat resolution (binary-only)
  runtime/                    bundled Lua plugins + init.lua scaffolding
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kohei-Wada/ttymap](https://github.com/Kohei-Wada/ttymap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
