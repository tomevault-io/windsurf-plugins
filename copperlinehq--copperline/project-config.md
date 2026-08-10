---
trigger: always_on
description: Guidance for AI coding agents driving Copperline, a cycle-driven Amiga emulator
---

# AGENTS.md

Guidance for AI coding agents driving Copperline, a cycle-driven Amiga emulator
(OCS/ECS/AGA) written in Rust. CLAUDE.md is a symlink to this file.

The most useful property for an agent: the emulated core is deterministic and
independent of wall-clock pacing. Headless runs are unthrottled, reproducible
byte-for-byte, and need no display or audio device, so the reliable way to
verify anything (a program you are developing, a config, a regression) is a
headless run with a scheduled screenshot, not a manual window session.

## Maintainer-local instructions

If `AGENTS.local.md` or `CLAUDE.local.md` exists in the repository root, read
it before starting work. Those files are gitignored, carry maintainer- and
machine-specific instructions, and take precedence over this file.

## Building and configuring

```sh
cargo build --release    # debug builds are far too slow for emulation use
```

Copperline bundles the open-source AROS boot ROM, so it boots with no config
file and no Kickstart at all. Configuration is a TOML file
(`copperline.example.toml` is the commented reference companion to
`docs/guide/configuration.md`), and the common machine knobs are also CLI
flags layered on top: `--model A500|A1200|CD32|...`, `--chipset OCS|ECS|AGA`,
`--cpu 68000..68060`, `--chip`/`--fast`/`--slow` memory sizes,
`--floppy-drives N`. A bare ROM path (only) is accepted positionally; disk
images go in via `[floppy.df0] path` in the config or `--insert-disk-after`:

```sh
./target/release/copperline --model A1200 --fast 8M KICK31.ROM \
  --insert-disk-after 0 df0 game.adf
```

Running with no arguments at all opens an interactive launcher window; any
flag suppresses it, so headless invocations never block on it.

WHDLoad game packages boot directly -- `--whdload game.lha` (or a directory
holding a `.slave`) stages a boot volume around the real WHDLoad program,
derives the machine from the slave header, and persists saves per game
(`docs/guide/whdload.md`). It needs the support archives fetched once by
`tools/fetch-whdload.sh` and, for real compatibility, Kickstart images via
`[whdload] kickstarts`; it composes with every headless flag below.

## Headless verification

All `SECS` timestamps below are absolute *emulated* seconds, not wall-clock.
Full reference: `docs/guide/headless.md`.

```sh
# Emulate 30s, save the framebuffer as PNG, exit.
./target/release/copperline --config my.toml --noaudio \
  --screenshot-after 30 /tmp/out.png

# Dump 120 consecutive rendered frames starting at 24s.
./target/release/copperline --config my.toml --noaudio \
  --dump-frames /tmp/frames --dump-start 24 --dump-count 120
```

Audio: `--noaudio` runs silent; `--audio-wav PATH` captures the mixed output
as a WAV in emulated time instead of playing it.

Guest clock: `--rtc-time "2005-03-18 01:58:29"` (Unix seconds also accepted)
fits a battery clock seeded to that instant, ticking in emulated time -- the
guest boots to the same deterministic time on every run, which is how to
test time-dependent guest software (TOTP vectors, date logic).
`--rtc-frozen` pins it to the seed exactly.

## Scripted input

Input is scheduled at emulated timestamps and composes with screenshots and
frame dumps to drive menus, loaders, and games deterministically. All flags
repeat.

| Flag | Effect |
|---|---|
| `--press-after SECS KEY` | Press and release an Amiga key (~100 ms hold) |
| `--key-after SECS KEY MS` | Hold a key for exactly MS milliseconds |
| `--click-after SECS BUTTON MS [PORT]` | Mouse button (`left`/`right`/`middle`) for MS ms (default port 1) |
| `--joy-after SECS BUTTON MS [PORT]` | Joystick / CD32-pad control (`up`/`down`/`left`/`right`/`red`/`blue`/...) (default port 2) |
| `--mouse-after SECS DX DY [PORT]` | Relative mouse motion (default port 1) |
| `--mouse-to-after SECS X Y [PORT]` | Steer the pointer to screen pixel (X, Y) via sprite 0 (default port 1) |
| `--pot-after SECS X Y [PORT]` | Analogue stick/paddle position, 0-255 per axis (default port 2) |
| `--insert-disk-after SECS DFN PATH` | Insert a disk image into `df0`..`df3` |
| `--insert-cd-after SECS PATH` | Swap the CD image in the machine's CD drive (CDTV/CD32/SCSI CD-ROM) |
| `--script FILE` | Same directives from a file, one per line, no leading dashes |
| `--record-input PATH` | Record all machine-bound input as a replayable script |

`KEY` is a raw key code (`0x45`) or a name (`ctrl`, `f1`, `esc`, letters,
digits). A session played by hand under `--record-input` (or Cmd+Shift+R /
Alt+Shift+R in the window) replays deterministically via `--script`.

Either controller port takes any device -- `[input] port1/port2` in the
TOML, or `--port1`/`--port2` (`mouse`/`joystick`/`cd32`/`analogue`/`none`;
default mouse + joystick, CD32 pad on the CD32 profile). The scripted-input
flags' optional trailing `PORT` token (`1` or `2`) aims an event at either
port; omitted, each flag keeps its traditional port, so existing scripts
are unchanged.

## Save states

`--save-state-after SECS PATH` snapshots the whole machine;
`--load-state PATH` resumes it. A resumed run is byte-identical to an
uninterrupted one. Pay a long boot/loading sequence once, then iterate from
just before the scene of interest:

```sh
./target/release/copperline --config my.toml --noaudio \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CopperlineHQ/Copperline](https://github.com/CopperlineHQ/Copperline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
