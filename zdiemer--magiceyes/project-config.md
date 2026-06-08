---
trigger: always_on
description: Run Game Park Holdings handheld games — **GP2X (F100/F200)**, **GP2X Wiz**,
---

# magiceyes — context for Claude

Run Game Park Holdings handheld games — **GP2X (F100/F200)**, **GP2X Wiz**,
**GP2X Caanoo** — on a PC, including DRM-locked commercial titles. Named for the
MagicEyes SoCs (MMSP2 in GP2X, Pollux in Wiz/Caanoo).

This file is the working brain: architecture, status, the reusable gotchas, the dev
environment, and where the (large, un-committed) assets live. Deep per-title war-stories
live in the memory files (see `MEMORY.md`) and `host/*/README.md`; read `README.md`
(user-facing) and `TODOS.md` (roadmap) too.

## Status (what works right now)

- **Wiz** — fully working via qemu+shim: **Deicide 3** (commercial, Inka DRM) and **Cave
  Story / NXEngine**. On the native Windows engine: **Her Knights**, **Deicide 3**,
  **Patissier** (EABI) boot to render+audio. *Open:* Her Knights BGM is radio static
  (8-bit sound bank corrupt before our SDL layer). See `wiz-titles-revival`.
- **GP2X** — **Payback** (commercial, static) playable end-to-end @ 30fps with video,
  audio, input, native threads, no crash — on both the qemu backend (`host/qemu/`) and
  the native Windows engine (`bin/magiceyes.exe`). More static titles render on the
  native engine: **Blazar, Quartz2, Vektar, Knight Lore**; dynamic: **Odonata, Wind &
  Water**. See `payback-loading-deadlock`, `windows-bundle-next`,
  `gp2x-static-titles-and-reload-crash`, `dynamic-gp2x-games-unicorn`.
- **Caanoo** (Pollux) — **Propis, Rhythmos, Liar** all run and render (software
  GLES1.1/EGL shim). Remaining: Rhythmos AVI-video background. See
  `caanoo-gpu-emulation`.

*Open issue (Windows-only):* hot-reloading twice to different games after a
memory-heavy first game hard-crashes (fork/Windows teardown; clean under Linux+ASan).
See `gp2x-static-titles-and-reload-crash`.

## Two backends (core design)

### 1. qemu-user + fake-SDL shim → Wiz (and any dynamic-libSDL title)
Wiz `.gpe` are EABI/glibc-2.3.6 ELF that dynamically link `libSDL-1.2.so.0`. Run under
`qemu-arm-static -L <wiz-rootfs>` and **replace `libSDL` with our own**
(`guest/src/fakesdl.c`) rendering into a `/dev/shm` framebuffer; native SDL2 viewer
(`host/viewer.c`) shows it + feeds input. DRM stubbed (`guest/src/drmstub.c`). Linux/WSL
only.

Shim gotchas (each was a real bug):
- SDL 1.2 **pre-silences the audio callback buffer** every callback (`memset` to
  `spec.silence`); games `SDL_MixAudio` onto silence. Not doing this = big distortion.
- `SDL_BuildAudioCVT`/`ConvertAudio` must really convert (U8→S16, resample, downmix).
- `SDL_Flip` must **frame-cap ~60fps** (real GP2X SDL_Flip blocks vsync; ours is instant
  → games run ~80× too fast). `FAKESDL_FPS` env.
- `SDL_LoadBMP_RW` must handle 1/4/8/24/32-bit BMP (GP2X art is mostly 4-bit + 1-bit).
- DRM: Inka "NED" `getserial` reads handset serial from `/dev/i2c-0`; no device → bails
  to `gp2xmenu`. Our stub libinkadrm/libdrmcode return success. Deicide/Caanoo/Liar
  assets are PLAINTEXT.
- GINGE is useless on PC (framebuffer/Pandora-host-locked, closed core) — hence our shim.
- `tools/extract_dat.py`: Deicide's `.dat` is a plaintext packed archive (fixed 140-byte
  header/entry; filename cstr, size u32 @+132, data @+140). Must extract or audio is garbage.

### 2. Unicorn engine → GP2X (static) + native cross-platform goal
GP2X `.gpe` are **GPEComp** self-extractors that decompress to a **statically-linked**
binary → no dynamic linker → `LD_PRELOAD` can't intercept. So GP2X needs syscall-level
emulation. `host/engine/` (split from the original `host/unicorn/me_unicorn.c`) is a
portable `qemu-user`-equivalent: **forked-Unicorn ARM CPU + ELF loader + Linux-ARM
syscall shim + GP2X hardware emulation**, presenting to the same shm viewer. This is the
path to native Windows/macOS/Linux binaries (no qemu/WSL). It also now has a dynamic-ELF
loader + rootfs path for Wiz/Caanoo titles, and an offline GPEComp decompressor
(`host/engine/gpecomp.c`).

The **qemu-user backend** (`host/qemu/`) is the verified-fast GP2X reference (full TCG
chaining + native threads); the Unicorn engine is being brought to parity and is the
shipping cross-platform path. See `cross-platform-fork-unicorn-plan`.

## Repo layout

```
guest/   src/{fakesdl.c, fakegles.c, drmstub.c, gp2xshm.h}  build_guest.sh  (ARM)
host/    viewer.c  build_viewer.sh                            (native SDL2 viewer)
host/    common/{gp2x_device.c, gp2x_device.h}               (engine-agnostic GP2X model)
host/    engine/{loader,elf,mem,devices,syscalls,gpecomp,guard,...}.c  (native engine)
host/    unicorn/{me_unicorn.c, build.sh}                     (original monolith / fallback)
host/    qemu/{gp2x.c, gp2x.h, apply_gp2x.py, build_qemu.sh, run-gp2x-qemu.sh, README.md}
host/    win/{stage_rootfs*.sh, build_*_win.sh, compat/, posix_compat.c, README.md}
tools/   extract_dat.py  un-gpecomp  gp2x/{decomp_*.sh, play.sh, ...}
README.md  TODOS.md  CLAUDE.md  .gitattributes  .gitignore
bin/     (build outputs, gitignored)
```
`gp2xshm.h` is the shm contract (RGB565 framebuffer + button bitmap + audio ring + touch),
shared by the shims, viewer, and engine.

## Build & run

**Wiz path (qemu+shim, Linux/WSL):**
```sh
MAGICEYES_SDK=<GPH SDK>  guest/build_guest.sh   # builds libSDL/libinkadrm/libdrmcode (ARM)
host/build_viewer.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zdiemer/magiceyes](https://github.com/zdiemer/magiceyes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
