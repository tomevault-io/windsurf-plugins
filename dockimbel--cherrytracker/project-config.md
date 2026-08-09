---
trigger: always_on
description: > **FINAL — DO NOT MODIFY (user-approved 2026-06-11):** the VU meters and the
---

# CherryTracker — notes for future Claude sessions

> **FINAL — DO NOT MODIFY (user-approved 2026-06-11):** the VU meters and the
> Spectrum Analyzer rendering + calibration (`vu-bar`, `grad-bar`,
> `build-spectrum`, `build-chan-vu`, the `SPEC-*` constants, ballistics, colors,
> flat bezel bands). Any change there needs an explicit user request.
> **Approved tweak (2026-06-12):** the spectrum peak caps now HOLD at a new peak
> for `SPEC-CAP-HOLD` frames (30 ≈ 0.5 s @60fps) before resuming the *unchanged*
> 3 px/frame fall — hardware peak-hold ballistics (the hold is the "lag"). This
> is intentional; don't "fix" it back to an immediate fall.

A playback-only ProTracker MOD player: **libxmp** (decode) → **SDL3** (audio) →
**Red/View** Draw UI. One compiled `.red` with embedded Red/System FFI.

## Build / run

- **Imports are EXTENSIONLESS and inlined** (`#import [ "libxmp" cdecl [...] ]`, `"SDL3"`) — no single-use `#define` macros for library names (user rule). The toolchain decides the link mode at the command line: **shared (.dll) by default**, **static (.lib archives) with `--static`**.
- Main source is **`player.red`**; the dev binary `player.exe` stays LOCAL (gitignored), the published release binary is **`CherryTracker.exe`** (in the repo — produced with `-o`).
- Dev build (dynamic, console subsystem): `D:\DevAI\Tools\redc.exe -c player.red` → `player.exe` — needs `libxmp.dll` + `SDL3.dll` + `libRedRT.dll` alongside. Extensionless PE import names resolve because the Windows loader appends `.dll`.
- **Release build (static, self-contained, GUI subsystem) — the verified command (user's up-to-date `Tools\redc.exe`; mine may lack `-s`):**
  ```
  ..\tools\redc.exe -r -s -t Windows -o CherryTracker.exe player.red
  ```
  `-s` switches the extensionless imports to the staged static archives; `-t Windows` drops the console for the shipping exe (dev builds keep it).
- Static archives staged in the project: **libxmp.lib** (clang-cl x86: `cmake -S libxmp -B build-static -A Win32 -T ClangCL -DBUILD_STATIC=ON` → target `xmp_static` → copy `build-static\Release\libxmp-static.lib` to `libxmp.lib`) and **SDL3.lib** (from `..\sdl-red\`).
- Headless smoke test: `redc -r -o test-play test-play.reds` then `test-play.exe` (plays ~8 s, prints pushed/queued).

## Architecture

- `xmp.reds` — libxmp cdecl `#import` + offset-based readers for `xmp_frame_info` / `xmp_module`. R/S `struct!` can't express the embedded `channel_info[64]` array, so fields are peeked by byte offset (`peek-i32`/`peek-u8`); offsets verified vs `include/xmp.h` @ 4.7.0 on x86. Modules load via `xmp_load_module_from_memory` (Red `read/binary` → `binary/rs-head`), avoiding path-encoding issues.
- `audio.reds` — minimal SDL3 (`SDL_Init` / `SDL_OpenAudioDeviceStream` / `Put` / `GetQueued`), S16 / 2ch / 44100, manual feed. SDL runs its own audio thread; we just keep its queue topped up.
- `player.red` — `#system-global` includes + player state + R/S helper funcs (pump / sync / ring) + `routine!` bridges (`pt-*`), then the Red Draw renderer + View loop.
- **The render path is ZERO-ALLOCATION (do not regress it).** Baseline cost of the naive draw-block-per-frame style was **10 GC runs/sec** (~23 MB/s of garbage, measured with a `-r -d` build — debug runtime prints one `runs: N … mark/sweep ms` line per collection); after the rework it is **0 GC runs over 30 s of playback**. The rules:
  - ONE persistent `frame-blk` is installed as `canvas/draw` ONCE; `render` does `clear frame-tail` (keeps the `anti-alias`/`scale` prefix) and re-appends every layer. `clear` keeps capacity → steady-state appends never allocate. Repaint is driven by the ownership events of the mutations themselves (no `show`, no per-frame `canvas/draw:` assignment).
  - Every helper is an `emit-*` that appends into `out` via **`compose/into [...] tail out`** — no intermediate blocks, parens produce immediates (pair!/tuple!/integer!). Never return fresh blocks / `make block!` / plain `compose` in the frame path.
  - **No string is ever built during a frame.** Static text comes from tables built once at startup (`note-strs`, `hex-strs`, `row-strs`, `chw-strs`/`chn-strs`, `state-tags`); live values go into dedicated per-slot reused buffers (`pos-buf`, `el-buf`, …) via `clear` + `pad2-into`/`pad3-into`/`fmt-time-into` (`append buf <int>` forms digits without garbage). Each draw slot needs its OWN buffer (the frame block holds references). Pattern cells use the grow-on-demand `cell-pool`. Load-time-only strings (`name-cache`, `file-line`, truncations) are prepared in `load-file`, never per frame.
  - `measure-*` / `text-size` (face-facet writes) are first-tick only — `emit-btn-glyph` reads the cached `icon-szs`.
  - No manual `recycle` anywhere — with clean frames it only adds pauses.
  - Per-frame `foreach` is avoided in emitters (allocates its context each call) — indexed `while` walks instead.
  - **Verifying render refactors:** dump `mold canvas/draw` at a deterministic tick (idle frame + a forced loaded/paused frame with seeded `vu-level`/`spec-level`/`spec-peak`, forced `view-pat`/`view-row`, a pressed button) from the old and new builds and byte-diff — proves the emitted Draw stream identical without pixel comparison.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dockimbel/CherryTracker](https://github.com/dockimbel/CherryTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
