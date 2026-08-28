---
trigger: always_on
description: Static recompilation of **Taiko no Tatsujin** (Bandai Namco, System 357 arcade
---

# TaikoRecomp — agent guide

Static recompilation of **Taiko no Tatsujin** (Bandai Namco, System 357 arcade
board = PS3 hardware, title `SCEEXE001`, S111 "Green") to native Linux and
Windows executables. SDL3 provides the portable graphical host. Uses the
[ps3recomp](https://github.com/sp00nznet/ps3recomp) framework (git submodule at
`ps3recomp/`). No game binaries/data in the repo — the dump lives in `game/`
(untracked).

## How it works

- `game/EBOOT.elf` — the game's PPU ELF, straight out of `unfself.py`. It is
  never patched: the Green dongle/VU bypass is applied to the lifted code by
  `tools/apply_recomp_patches.py` instead.
- `src/recomp/ppu_recomp_*.cpp` — ~8 chunks of lifted PPU code (millions of
  lines, generated; each guest function is `func_00XXXXXX(ppu_context*)`).
  Grep here to inspect guest code by address.
- `src/gen/` — generated HLE stubs + NID table.
- `ps3recomp/runtime/` — PPU loader/VM (flat 4 GB guest space at `vm_base`,
  demand-committed via VEH on Windows), HLE dispatch, lv2 syscalls, cellFs VFS.
- `ps3recomp/libs/video/` — RSX HLE: `cellGcmSys.c` (FIFO drain, flips,
  offsets), `rsx_commands.c` (NV4097 method decode → `rsx_state`), the portable
  recorder/batch model, and `rsx_sdl_gpu_backend.c` (SDL_GPU execution,
  offscreen RTs, shader translation, pipeline cache, and presentation).
- Title-specific shims in `src/`: `taiko_usio.cpp` (virtual PS3A-USJ arcade I/O
  board + backup SRAM + network-state spoof), `taiko_sail.cpp` (cellSail
  lifecycle so the movie wrapper doesn't hang), `taiko_net.c` (DNS loopback),
  `taiko_init.c`.
- `ghidra_out/` — `functions.json`, `symbols.json`, `strings.json` from Ghidra:
  the map for symbolizing guest addresses (OPD pointers, string refs).

Frame flow: guest writes the GCM FIFO → host vblank-ticker thread drains it at
~4 ms (`cellGcm_rsx_process_fifo`) → the recorder snapshots backend-neutral,
owning render batches → the SDL main thread executes them with SDL_GPU and
presents. Display clears and flips delimit complete batches without exposing a
partially recorded frame.

## Build

Two targets. `build/` is the MinGW/Windows build run under Wine; `build-linux/`
is the native Linux build. Both read their dependencies from `third_party/`, so
either build directory can be deleted and recreated freely.

```sh
scripts/setup_sdl_gpu_mingw.sh # once: pinned SDL3 + shadercross + DXC target bundle
scripts/setup_sdl_gpu_linux.sh # once: the same bundle for the native build
scripts/build_ffmpeg_mingw.sh # once: pinned minimal static ATRAC3plus decoder

# Windows (via mingw-w64 + Wine)
cmake -S . -B build -G Ninja -DCMAKE_TOOLCHAIN_FILE=mingw-w64.cmake -DCMAKE_BUILD_TYPE=Release
cmake --build build          # incremental; full link of the lifted code is slow

# Native Linux
cmake -S . -B build-linux -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DTAIKO_RSX_BACKEND=sdl_gpu -DTAIKO_INPUT_BACKEND=sdl3 \
  -DTAIKO_AUDIO_BACKEND=sdl3 -DTAIKO_INPROCESS_ATRAC=ON \
  -DTAIKO_EMBED_PPU_IMAGE=OFF
cmake --build build-linux
```

**Parallelism is capped on purpose** (`TAIKO_COMPILE_JOBS`, default 4, via a
ninja job pool). Each lifted chunk is a 600k–800k line TU needing several GB at
`-O2`; ninja's default of one job per core meant 16 concurrent multi-GB compiles,
which exhausts RAM and wedges the machine rather than merely being slow.

Budget **~3 GB per job** and set it from available RAM, not core count:

```sh
cmake -S . -B build -DTAIKO_COMPILE_JOBS=8 ...   # ~24 GB peak, needs 32 GB+
```

The default stays at 4 so builds run unattended. **Agents must use 4 on this
development host**: an eight-job lifted-code rebuild made the desktop
unresponsive even with ample nominal RAM. Configure native build directories
with `-DTAIKO_COMPILE_JOBS=4`, and invoke cross-build scripts as
`TAIKO_COMPILE_JOBS=4 ./scripts/build_rpi_arm64.sh`. Never start a second build
while one is active. The limit is a cache variable — pass it at configure time,
or reconfigure an existing build directory before expecting it to apply.

Changing the chunk count (a re-lift) needs a **re-configure**, not just a build —
`file(GLOB)` is evaluated at configure time.

Stale objects used to be a second trap here: ninja compares mtimes, and a
freshly lifted chunk can look older than the object built from the previous
lift, so the link fails with undefined references to `func_*` (a re-lift moves
functions between chunks). `apply_recomp_patches.py` now stamps every chunk to
the current time, so this resolves itself. If you ever lift *without* running
it, `rm -f build*/CMakeFiles/taiko_boot.dir/src/recomp/*.obj` is the manual
fix.

## Run

```sh
./run-taiko.sh               # logs to build/taiko.log (TAIKO_CONSOLE_LOG=1 for stdout)
```

The script sets everything needed. Notable pieces:

- `TAIKO_GPU_DRIVER=vulkan` makes the Windows SDL_GPU build use Wine's Vulkan
  path directly. Native Windows leaves the variable unset so SDL can select
  `direct3d12` or `vulkan`. The old vkd3d-proton override is obsolete.
- `PS3_VFS_ROOT` — host dir the PS3 mount points map into (`game/vfs`).
  **All of the game's mount points must exist there**, not just `data`. The
  title also uses `/cache`, `/hash`, `/install`, `/logs` and `/updates`, all of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LucaSilva-r/TaikoGreenRecomp](https://github.com/LucaSilva-r/TaikoGreenRecomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
