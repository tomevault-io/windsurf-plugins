---
trigger: always_on
description: These instructions apply to the whole repository.
---

# Repository Instructions

## Scope

These instructions apply to the whole repository.

## Debugging on real MiSTer hardware

If you are bringing up or debugging the core on a **real MiSTer / DE10-Nano**
(not just the Verilator sim), read **`docs/MISTER_HARDWARE_DEBUGGING.md`** first.
It covers the Quartus build/program/probe loop, JTAG in-system probes
(`rtl/dbg_min.sv` + `scripts/cpu_state.tcl`), SignalTap, the MiSTer web UI /
Remote API for screenshots, ROM-loading quirks, and the sim-vs-hardware
differences behind "works in sim, fails on hardware" bugs.

## Workflow

- Keep work on the newest active branch for this effort. Check `git status --short --branch` before committing.
- Do not check in the local MAME checkout, ROM zips, disk images, generated simulator logs, screenshots, or build outputs.
- Use `rg`/`rg --files` for code search.
- Prefer small commits that leave the boot-debug state reproducible.

## Verilator

Build from `verilator/`:

```sh
make
```

Run the current direct floppy comparison from `verilator/`:

```sh
./obj_dir/Vemu --headless --no-cpu-trace --no-via-debug \
  --floppy0 ../releases/Disk605.dsk --stop-at-frame 300
```

Use `--periph-debug` only for short focused runs; it writes `verilator/periph_debug.log`.
Generated logs such as `cpu_trace.log`, `via_debug.log`, `periph_debug.log`,
`ram_debug.log`, and temporary `sim_*.log` files should stay untracked.
Use `--scsi-debug` for focused NCR5380 transaction traces and
`--nubus-video-full-debug` only when the VRAM/register/RAMDAC write stream is
needed; both write to stderr and can be noisy on long runs.

## MAME Reference Runs

Use the local ignored checkout at `mame/`. Do not use plain `./mame macii` for
video-card comparisons: MAME's default slot layout installs `mdc824` in slot 9,
while this core models the Apple Macintosh II High Resolution Video Card in slot
E. Match the core by removing slot 9 and installing `m2hires` in slot E:

```sh
cd mame
SDL_VIDEODRIVER=dummy ./mame macii \
  -rompath roms -video none -sound none -nothrottle -skip_gameinfo \
  -nb9 "" -nbe m2hires -scsi:6 "" -flop ../releases/Disk605.dsk \
  -autoboot_script ../tools/mame/macii_frame_probe.lua
```

For frame-limited probes, pass `MAME_STOP_FRAME` and related probe environment
variables before the command, for example:

```sh
MAME_FRAME_INTERVAL=20 MAME_STOP_FRAME=120 SDL_VIDEODRIVER=dummy ./mame macii \
  -rompath roms -video none -sound none -nothrottle -skip_gameinfo \
  -nb9 "" -nbe m2hires -flop ../releases/Disk605.dsk \
  -autoboot_script ../tools/mame/macii_frame_probe.lua
```

For register snapshots around a ROM PC range, use:

```sh
MAME_MIN_FRAME=260 MAME_FRAME_INTERVAL=20 MAME_STOP_FRAME=520 SDL_VIDEODRIVER=dummy ./mame macii \
  -rompath roms -video none -sound none -nothrottle -skip_gameinfo \
  -nb9 "" -nbe m2hires -scsi:6 "" -flop ../releases/Disk605.dsk \
  -autoboot_script ../tools/mame/macii_pc_region_probe.lua
```

For the current ROM wait-helper comparison, use the same card and no default
SCSI hard disk:

```sh
MAME_STOP_FRAME=900 MAME_FRAME_INTERVAL=80 MAME_MAX_PRINT=160 SDL_VIDEODRIVER=dummy ./mame macii \
  -rompath roms -video none -sound none -nothrottle -skip_gameinfo \
  -nb9 "" -nbe m2hires -scsi:6 "" -flop ../releases/Disk605.dsk \
  -autoboot_script ../tools/mame/macii_wait_probe.lua
```

For the low-memory delay calibration setup, use:

```sh
MAME_STOP_FRAME=120 MAME_FRAME_INTERVAL=20 MAME_MAX_PRINT=300 SDL_VIDEODRIVER=dummy ./mame macii \
  -rompath roms -video none -sound none -nothrottle -skip_gameinfo \
  -nb9 "" -nbe m2hires -scsi:6 "" -flop ../releases/Disk605.dsk \
  -autoboot_script ../tools/mame/macii_calib_probe.lua
```

The local MAME ROM setup expected for the matched card is:

- Mac II system ROM in `mame/roms/macii.zip`.
- High Resolution Video Card declaration ROM at `mame/roms/nb_m2hr/341-0660.bin`.
- `releases/341-0660.bin` / `releases/boot1.rom` should match SHA1
  `37c59f38ae34021d0cb86c2e76a598b7e6077c0d`.

## Current Debug Focus

RAM sizing, early ASC, the initial IWM probe, and the no-media drive-queue node
now match the matched-card MAME run closely enough that they are not the current
blocker.

For the current no-media baseline, run MAME without a floppy image and with the
same slot-E card:

```sh
MAME_STOP_FRAME=800 MAME_FRAME_INTERVAL=80 SDL_VIDEODRIVER=dummy ./mame macii \
  -rompath roms -video none -sound none -nothrottle -skip_gameinfo \
  -nb9 "" -nbe m2hires -scsi:6 "" \
  -autoboot_script ../tools/mame/macii_bootvars_probe.lua
```

MAME stays in the no-media drive-queue loop at `PC=$408061F2` with
`$030A=$2F70`, `$2F70.next=0`, `$2F70+$06=0001`, and `$2F70+$08=FFFB`.
Verilator now reaches the same queue state after fixing the external floppy
connector to report no installed drive by default. MAME's `add_35_nc` external
connector uses a `nullptr` default device.

The `$09FA-$0A02` low-memory words are `TempRect`/`OneOne` QuickDraw scratch
state, not a SCSI boot variable. A short Verilator sample showed
`$09FA/$09FC/$09FE=FFFD/FFFD/01E3`, but a longer run to the actual SCSI timeout
shows the same final values as MAME:

```text
$09FA=0800 $09FC=0000 $09FE=0100 $0A00=0283 $0A02=0001
```

The current no-media comparison needs one important caveat. Verilator executes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danifunker/lbmactwo_MiSTer](https://github.com/danifunker/lbmactwo_MiSTer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
