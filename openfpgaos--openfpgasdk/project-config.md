---
trigger: always_on
description: Handles stay valid across voice stealing — `of_mixer_handle_active(h)`
---

# CLAUDE.md — openfpgaOS SDK and game ports

Guidance for AI coding agents (Claude Code reads this file; Codex reads
`AGENTS.md`, which is the same file) working in this SDK or in a game-port
checkout built from it.

## What this is

SDK for **openfpgaOS** — a bare-metal RISC-V game runtime (VexiiRiscv
rv32imafc @ 100 MHz, 64 MB SDRAM) on two platforms: **Analogue Pocket**
and **MiSTer** (DE10-Nano / SuperStation One). One app `.elf` runs
unchanged on both; everything platform-specific arrives at runtime via
the capability/services tables (`of_get_caps()`, auxv). Apps are trusted,
single-process, no MMU — they run close to the hardware.

Game ports (Duke3D, Doom, Quake, QuakeSpasm, Quake2, Diablo, ScummVM, …)
are **forks of this SDK**: same root layout, the game lives in
`src/<name>/` next to SDK-owned `src/sdk/` and `src/apps/`.

## Build, test, deploy

```bash
make build CORE=<name>     # build a custom core (or cd src/<name> && make)
make test  CORE=<name>     # desktop SDL2 build (./app_pc) — fastest iteration
make copy  CORE=<name>     # deploy to Pocket SD card
make copy  CORE=<name> TARGET=mister   # network-push to a MiSTer (MISTER_IP env, default mister.local)
make debug CORE=<name>     # UART push + console stream (Pocket + DevKey only)
```

Do not edit SDK-owned directories: `src/sdk/`, `src/apps/` (except the app
you're working on), `scripts/`, `runtime/` (build artifacts synced from the
openfpgaOS repo), `src/sdk/platforms/mister/fatfs/` (vendored, pristine).

## Porting ground rules

- **Never hardcode addresses or sizes** — read `of_get_caps()` (fb_base,
  heap, sample pool, gpu_base, cpu_freq_hz). The memory map is the
  portability anchor; the caps table is how you stay on it.
- **Gate hardware on feature bits**, not platform ids, wherever possible:
  `of_has_feature(OF_HW_GPU_PERSP)`, `of_has_feature(OF_HW_ANALOGIZER)`, …
  Use `of_get_caps()->platform_id` (`OF_PLATFORM_POCKET/MISTER/SIM`) only
  for genuinely platform-shaped behavior.
- **Pocket-only**: `make debug` (PHDP/UART), `interact.json` menus,
  Analogizer/SNAC. On MiSTer `of_analogizer_enabled()` returns 0.
- Files: register names once (`of_file_slot_register(slot, "game.grp")`),
  then standard `fopen`/`fread`. Filenames ≤ 23 chars (registry limit, both
  platforms). POSIX `lseek` must use the riscv32 5-arg `_llseek` convention
  (see `src/sdk/include/of_posix.c`).
- Keep a CPU fallback for every GPU path and keep the desktop build
  (`make test`) green — it is the fast correctness reference.

### Reference ports (read these before inventing a new pattern)

| Port | GPU | Audio | Input | Key files |
|---|---|---|---|---|
| Duke3D | affine span groups (Build engine) | mixer groups + MIDI | pad + P2 + kbd + mouse | `src/duke3d/d3d_gpu.c`, `d3d_audio.c`, `Engine/src/display_of.c` |
| Doom | affine spans + translucency (fuzz) | mixer channels | pad + analog | `src/doom/cdoom/doom/r_gpu.c`, `shim/i_sdlsound.c`, `shim/i_input.c` |
| Quake | perspective span groups | stream + 2-voice CD music | pad + analog | `src/quake/engine/vid_of.c`, `cd_of.c`, `snd_of.c`, `in_of.c` |
| QuakeSpasm | affine + persp, palookup cache | mixer groups | kbd/mouse shims, deadzone | `src/quake/gpu/qs_gpu.c`, `shim/snd_of.c`, `shim/in_of.c` |
| Quake2 | param span lists (Z modes) | mixer SFX + streamed music | pad | `src/quake2/openfpga/of_emit_q2.c`, `snd_of.c`, `input_of.c` |

Minimal working examples in this repo: `src/apps/gpudemo` (GPU),
`src/apps/wavdemo`/`moddemo`/`mididemo` (audio), `src/apps/sdldemo` (SDL2).

## GPU — optimizing the renderer

The GPU is an **asynchronous indexed-color span rasterizer** built for
BUILD/Doom/Quake-style software renderers. The CPU stages command words in
a cached SDRAM scratch buffer; a doorbell DMA pulls them into the GPU's
16 KB ring; fences report completion. There is no per-command MMIO path —
**batching is the design, not an optimization**.

### Integration pattern

- `of_gpu.h` contains **static mutable state — include it from exactly ONE
  translation unit** and export your own wrapper API to the rest of the
  game (every port does this: `d3d_gpu.c`, `r_gpu.c`, `vid_of.c`, `qs_gpu.c`).
- Init order: `of_get_caps()` → check `OF_HW_GPU_*` bits → `of_gpu_init()`
  → draw a probe span to a scratch framebuffer and verify the pixels →
  fall back to the CPU renderer on failure.
- Gate optional GPU features individually: `OF_HW_GPU_SPAN` is the
  baseline (always set); `OF_HW_GPU_PERSP`, `OF_HW_GPU_PARAM_SPAN_LIST`,
  `OF_HW_GPU_PARAM_SPAN_Z/ZTEST/Q29_SCALE`, `OF_HW_GPU_ALPHA`,
  `OF_HW_GPU_BILINEAR`, `OF_HW_GPU_VCOLOR` are not.

### Which span command to use

- `of_gpu_draw_affine_span_group()` — constant-z texture stepping:
  BUILD walls/floors, Doom columns/spans. Up to 8 lanes per group
  (per-lane fb/tex addr, count, s/t steps, 6-bit light, colormap_id);
  the SDK splits >4 lanes into 4-lane hardware chunks.
- `of_gpu_draw_persp_span_group()` (+ `_batch`) — perspective-correct
  spans (Quake world). Accumulate spans and submit groups; don't emit
  one group per span.
- `of_gpu_draw_param_span_list()` — the unified parametric command
  (affine/persp/solid attr modes, Z write/test, Q29 scale). Quake2's
  renderer sits on this. Gate on `OF_HW_GPU_PARAM_SPAN_LIST`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openfpgaOS/openfpgaSDK](https://github.com/openfpgaOS/openfpgaSDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
