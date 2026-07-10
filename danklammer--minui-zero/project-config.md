---
trigger: always_on
description: > **Name: MinUI Zero** (chosen 2026-07-01) — "drive every unused rail to zero." A fork of
---

# CLAUDE.md — MinUI Zero: performant/efficient MinUI fork for the TrimUI Brick

> **Name: MinUI Zero** (chosen 2026-07-01) — "drive every unused rail to zero." A fork of
> `shauninman/MinUI`. The name lives in `README.md`; internal files/identifiers are **not** renamed
> for branding (keeps upstream merges clean) — the `ZERO_*` env flags are the only Zero-branded symbols.

## What this is
A fork of MinUI focused on **performance through efficiency** on the TrimUI Brick
(`tg5040` platform, Allwinner A133P). The thesis: run each emulated system at the **lowest
CPU clock that still holds its target frame rate**, so the device stays cool and sips power.
This is *not* a feature fork — it's the opposite. NextUI is the feature-rich/GL fork; this
one is the distilled, runs-cold one.

## Scope — **tg5040 only** (TrimUI Brick + TrimUI Smart Pro)
This is a single-platform fork. The whole thesis is A133P-specific (cpufreq/OPP/thermal/PMIC),
and we only test on the Brick, so we **support `tg5040` exclusively**. Like NextUI, the other
MinUI platforms are frozen under `workspace/_unmaintained/` — present for history/upstream
merges, but **not built or supported** (`make` defaults to `PLATFORMS = tg5040`). `workspace/
macos/` stays as the zero-hardware dev/test platform (launcher build + harnesses), not a device.
Don't re-add other devices without doing that device's full bring-up (recon + per-SoC wiring).

## North star / non-negotiables
- **Cool + efficient is the whole point.** Every change should serve "lowest clock that holds
  frame rate." If a change adds heat, idle power, or resident memory without earning it, don't.
- **Default to MinUI's lean software (RGB565) render path.** Fix tearing the cheap way first
  (page-flip + double-buffer / NEON scalers, referencing MyMinUI). GLES is **benchmarked, not
  auto-rejected** — adopt only if it wins on *total-device* power/temperature, not CPU% (it
  keeps the GPU lit, which usually loses). See `docs/project-direction.md` §2.
- **"Minimal" describes the UX, not the implementation.** Substantial internal change is fine
  where it earns thermals, gameplay, frame pacing, suspend/save reliability, or crash
  resistance. User-facing features are still weight — no box art, WiFi/NTP, Pak Store,
  RetroAchievements, ambient-LED, overlays. Authoritative direction + roadmap:
  **`docs/project-direction.md`** (it supersedes this file where they differ).
- **Never overclock, never fabricate device values.** Real OPP steps / thermal-zone paths come
  from the hardware (`tools/brick-recon.sh`); query the OPP table at runtime. **Never use
  2.0 GHz** unless on-device evidence proves it's a stock (non-OC) operating point — default
  the cap to the highest *verified-stock* OPP. Until measured, use the **clearly-labeled
  assumptions** in `docs/thermal-governor-design.md` (safe by construction, not hidden guesses).

## Hardware (target platform = `tg5040`)
- **SoC:** Allwinner A133P, quad-core Cortex-A53. MinUI's tg5040 code drives it via the
  `userspace` cpufreq governor, writing kHz to
  `/sys/devices/system/cpu/cpu0/cpufreq/scaling_setspeed`.
- **Known reference clocks (read from tg5040 source, not assumed):** 600 / 1200 / 1608 / 2000
  MHz = menu / powersave / normal / performance. (2.0 GHz is a mild OC over the 1.8 GHz stock.)
- **GPU:** PowerVR GE8300 — irrelevant here (software render path).
- **Display:** 1024×768 IPS. **RAM:** 1 GB.
- The Brick and the TrimUI **Smart Pro** share the `tg5040` platform. (The plain Smart is
  `trimuismart` — different platform; ignore it.)

## Architecture — where things live (verify by grepping; line numbers are approximate)
- `workspace/all/minarch/minarch.c` — the libretro frontend + main run loop. **The governor
  tick goes here.** Already measures frame pacing around `GFX_flip` / `FRAME_BUDGET`.
- `workspace/all/common/api.c` — `GFX_flip` / `GFX_sync` / `PLAT_vsync` (frame pacing,
  `gfx.vsync = VSYNC_STRICT`); `PWR_*` (sleep/power); `SND_*` (audio).
- `workspace/all/common/scaler.c` / `scaler.h` — **our software RGB565 scalers.** This is the
  render path to keep/optimize.
- `workspace/all/common/api.h` — `CPU_SPEED_*` enum (~L298); `#define PWR_setCPUSpeed
  PLAT_setCPUSpeed`.
- `workspace/tg5040/platform/platform.c` — `PLAT_setCPUSpeed` (~L546, the freq write),
  `PLAT_blitRenderer` (~L367), `PLAT_flip` (~L373), `PLAT_setVsync`/`PLAT_vsync`.
- `workspace/tg5040/install/boot.sh` — sets `userspace` governor + clock at boot.
- `workspace/tg5040/cores/patches/` — per-core build patches (gpsp, mgba, pcsx_rearmed,
  picodrive, snes9x2005_plus, fceumm, gambatte, …). Start core tuning from these.
- `workspace/macos/` — **dummy desktop platform** (its own notes say so): builds the launcher
  on macOS under AddressSanitizer, no cores, stubbed input. Our zero-hardware playground for
  logic — useless for thermal/perf (no real clocks or heat).

Full three-way file map (incl. NextUI/MyMinUI equivalents): **`docs/architecture-map.md`**.

## Reference forks (git remotes — borrow freely)
This is a community fork scene — borrow and adapt code from any of these freely; keep
attribution as a courtesy. The only real friction is technical, not legal: NextUI modularized

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danklammer/MinUI-Zero](https://github.com/danklammer/MinUI-Zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
