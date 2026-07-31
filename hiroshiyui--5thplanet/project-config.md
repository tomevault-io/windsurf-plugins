---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

5thPlanet is an **accuracy-first** SEGA Saturn emulator in Rust. The Saturn has eight processors with tightly-coupled timing (2× SH-2 SH7604, MC68EC000, VDP1, VDP2, SCU + SCU-DSP, SCSP M68k + SCSP-DSP, SH-1 CD-block); the project is built up one chip at a time so the foundation stays solid. **Performance is explicitly subordinated to fidelity** — never introduce a JIT, dynarec, or "approximate cycle" shortcut.

The one deliberate exception is the **SH-1 CD-block, which is high-level-emulated (HLE), not low-level-emulated** — its CD-ROM firmware is undumped (on-die mask ROM) and half its job is an analog servo with no observable digital ground truth, so there's nothing to be cycle-accurate *against*. Like every Saturn emulator (MAME/Yabause/Mednafen), we model the host command interface + the buffer/filter/partition engine + the CD-ROM filesystem, reading sectors from a disc image. This is M7 (the HLE-exception decision is **ADR-0015**); see `doc/roadmap.md` and `crates/saturn/src/cd_block.rs`.

**M1–M8 are complete.** M1 (cycle-accurate SH-2 core), M2 (Saturn bus + dual SH-2 + event-driven scheduler), M3 (SCU + SMPC + VDP2 minimal + SCU-DSP + SDL3 scaffolding), M4 (BIOS boots to the SEGA splash — now pixel-matching MAME), M5 (chip-coverage build-out: VDP1 full plotter, MC68EC000 core, full VDP2 NBG/RBG compositor), M6 (SCSP slot/FM audio engine), M7 (the **CD-block**, high-level-emulated — see above — plus the **cartridge slot**), and M8 (**save states + battery-backed backup RAM**). M7's five HLE phases (disc-image loading + TOC/session, the buffer/filter/partition engine, the read pump + data transfer, the ISO9660 filesystem, disc authentication) plus the cartridge slot are done. M8 adds `Saturn::save_state`/`load_state` (bincode snapshot of the whole machine; external media referenced not embedded) and a hardware-faithful, host-persisted internal backup RAM. **M9 (frontend OSD) is ✅ complete (2026-06-11)** — a hand-rolled, software-composited in-window menu (ADR-0008): Esc opens it, with save/load slots, reset, eject/insert disc, **a disc-image browser ("Load Disc…" → navigate the filesystem, pick a `.cue`/`.iso`/`.ccd`, load + boot it)**, quit, and Settings submenus for Graphics (scale 1×–4×, fullscreen, renderer backend, + a **Shaders chooser** (None / CRT — the SDL_GPU CRT presenter has since landed, ADR-0019 / `jupiter/src/shaders/`, gated behind the off-by-default `gpu-presenter` build feature so normal builds don't surface it)), **Controller (per-port device assignment — keyboard / a specific game controller / Shuttle Mouse / analog 3D Control Pad — plus press-to-bind keyboard *and* per-button gamepad rebind; M13 Tier E superseded the old Off/1/2 mouse toggle)**, Region, Cartridge, and **BIOS (power-cycle into a sibling 512-KiB image, save files re-keyed)** — all persisted to a flat TOML-subset **config file** at `$XDG_CONFIG_HOME/5thplanet/jupiter.toml` (`jupiter/src/config.rs`; precedence CLI flag > config > autodetect). **Basic SDL3 GameController support is in** (hot-plug; fixed Xbox-style mapping A/B/C = X/A/B, X/Y/Z = Y/LB/RB, L/R = triggers, left stick = D-pad; controller navigates the OSD). **M13 Tier E input has since landed:** per-port device assignment (`4731c7a`, config `port1`/`port2`), the analog **3D Control Pad** (`6482bbd`/`6ee0ab6`, INTBACK ID `0x16`, savestate v16), and per-button gamepad rebind (`01aa486`, config `gpad_*` SDL tokens) — E-2c Mission Stick/wheel, E-3 multitap, and E-4 gun/keyboard are deferred pending test titles as their LLE oracle. **M10 is done: live physical disc + CDDA→SCSP** — a `SectorSource` trait (image or live drive), CD-audio BGM mixed into the SCSP, and optical-drive reads via the feature-gated `physdisc`/libcdio crate (ADR-0009). Still deferred from M7: the MPEG card and move/copy sector ops.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiroshiyui/5thPlanet](https://github.com/hiroshiyui/5thPlanet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
