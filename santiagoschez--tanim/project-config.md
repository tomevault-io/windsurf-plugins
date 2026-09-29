---
trigger: always_on
description: Guidance for AI coding agents (and humans) working on tanim: endless procedural screensaver animations for the terminal, written in Rust, also compiled to WebAssembly for the browser. See `README.md` for the user-facing side.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working on tanim: endless procedural screensaver animations for the terminal, written in Rust, also compiled to WebAssembly for the browser. See `README.md` for the user-facing side.

## Commands

```sh
cargo build --release                        # also builds the wasm module (see build.rs)
cargo test --release                         # unit tests
./target/release/tanim --check               # every animation headless at 1x1 up to 240x70, with cost per frame
./target/release/tanim --check dog,dungeon   # just some
./target/release/tanim --dump <name> WxH <frames> [warmup] [--seed n] [--zoom n]   # raw frames for scripts
./target/release/tanim --snapshot <name> WxH <frame>    # one frame as text
make install                                 # ~/.local/bin/tanim
make html                                    # web pages in html/
make videos                                  # WebM per animation in videos/ (needs Python + ffmpeg)
make streamdeck MODELS=neo                   # Stream Deck GIFs in streamdeck/<model>/
```

Before finishing any change: the build must have **zero warnings**, `cargo test --release` must pass, and `tanim --check` must run the touched animations without panicking. Look at the result too: render frames to PNG from `--dump` (the rasterizer in `scripts/video.py` can be imported for this) and check them by eye, because most bugs here are visual.

## Layout

| Path | What |
|---|---|
| `src/lib.rs` | Library root: `anims`, `canvas`, `rng`, `zoom` and the `Key` enum. Shared by the terminal binary and the web build. |
| `src/main.rs`, `src/term.rs` | Terminal binary: CLI parsing, frame loop, raw mode, key parsing (libc only). |
| `src/canvas.rs` | `Canvas` (cells), `Rgb` helpers and `Screen`, the diffing renderer that emits escape sequences. |
| `src/anims/` | One module per animation plus `mod.rs` with the `Animation` trait and the `catalog!` macro. |
| `src/zoom.rs` | Generic digital zoom and pan wrapped around every animation. |
| `src/export.rs`, `src/web.html`, `src/favicon.png` | `--export-html`: standalone pages embedding the wasm module and favicon (base64) and the canvas player. |
| `web/` | Separate crate (its own workspace) exposing the animations to JavaScript as plain `extern "C"` functions. |
| `build.rs` | Builds `web/` for `wasm32-unknown-unknown` and embeds it; without that target the build succeeds and the export is disabled. |
| `scripts/video.py` | Rasterizes `--dump` output and encodes WebM videos or Stream Deck GIFs with ffmpeg. |
| `.github/workflows/pages.yml` | On every push to `main`: brings the Stream Deck GIFs in the `gh-pages-assets` branch up to date, then rebuilds the web version with them at `streamdeck/` and publishes it to GitHub Pages. |

## How rendering works

- An animation draws into a `Canvas` of `w x h` terminal cells. Nearly all of them draw **half-block pixels**: each cell is `▀` with the foreground as the top pixel and the background as the bottom one, giving a `w x 2h` grid of square pixels. The usual pattern is a `Vec<Rgb>` of `w * 2h` pixels passed to `c.blit_pixels(&px)`; `c.pixel()` paints single ones. Text still works with `c.text()` after blitting (it keeps each cell's bottom pixel as background). Only `matrix` is glyph-based on purpose.
- `Screen::render` diffs against what the terminal already shows and sends only changed cells, wrapped in synchronized output (mode 2026). A small deadband skips color changes of 2 or less per channel. **Output bytes per frame are the real performance cost** (terminal CPU), far more than compute: keep static parts static, avoid per-pixel noise that changes every frame, and check the `B/f` column of `--check` before and after a change.
- Animations registered with `clear = module::BG` in the catalog get a transparent background: pixels exactly equal to that color are drawn as the terminal's default background. Keep empty pixels exactly `BG` (snap near-black fades to it), or they show as dark boxes on a transparent terminal.
- The frame loop runs each animation at its catalog fps and, when the terminal falls behind, steps without drawing (up to 5 steps) instead of going into slow motion.

## Adding or changing an animation

1. Create `src/anims/<name>.rs` with a `//!` header describing what it shows and how it works, and `pub fn new(w: usize, h: usize, rng: &mut Rng) -> Box<dyn Animation>`.
2. Register it in the `catalog!` macro in `src/anims/mod.rs` with its fps (usually 20 or 30), an English one-line description, and `clear = module::BG` if the background is a plain dark color.
3. Add a row to the catalog table in `README.md`.
4. It must survive **every size down to 1x1** (`--check` tests 1x1, 2x2, 7x3, 13x5, 80x24 and 240x70): guard empty ranges, `clamp` with min > max, divisions by zero and out-of-bounds indexing.
5. Use only the `Rng` passed in: no `std::time`, `std::process` or other OS calls inside `src/anims`, `canvas`, `rng` or `zoom`, because that code also runs as wasm. `Rng::new(seed)` must keep giving a stream per seed (`--seed` reproducibility is relied on by the scripts).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SantiagoSchez/tanim](https://github.com/SantiagoSchez/tanim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
