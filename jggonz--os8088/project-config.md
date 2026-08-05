---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

os8088: a Macintosh System 1-style GUI OS for the Intel 8086, written entirely in real-mode NASM assembly, booted from floppy. Pre-emptive multitasking, overlapping windows, serial mouse, a bottom dock, and loadable software packages that run as closable, multi-instance apps — all in 256KB of RAM. One binary drives VGA, Hercules or CGA, picked at boot.

**SPEC.md is the binding contract.** Every symbol name, register contract, constant, and data layout is pinned there. Update SPEC.md *before* changing any interface, not after.

## Commands

```
make          # build all four floppy images into build/
make run      # boot in QEMU with emulated serial mouse (1.44MB images)
make run-640  # same, as a maxed-out 640KB machine (-m 1M; QEMU/SeaBIOS can't boot below 1MB, int 12h caps at 640K anyway; SeaBIOS's EBDA makes it 639K)
make test     # boot headless with QMP socket at build/qmp.sock for scripted testing
make test ADLIB=1 # ...with an emulated AdLib at 388h, so the sound DRIVER
              # (SPEC.md 51.4) has something to attach to. SB16=1 likewise.
make test-snd # make test + PC speaker captured to build/snd.wav; verify with
              # tools/sndcheck.py (note: the wav holds speaker-ON time only, not
              # wall time - a silent boot yields an empty capture, and QEMU leaves
              # the RIFF sizes zeroed, which sndcheck.py absorbs)
make debug    # boot QEMU halted, waiting for gdb on :1234
make xt       # boot 360KB images on an emulated IBM PC/XT in 86Box
make xt-640   # same XT with a full 640KB RAM (vm/xt640/86box.cfg)
make xt-cga      # XT + real CGA card, 256KB (vm/xt-cga)
make xt-hercules # XT + real Hercules card, 256KB (vm/xt-hercules)
make 286         # 86Box AT clone: 286 @ 12.5MHz, 1MB, VGA (vm/286)
make 386sx       # 86Box Shuttle HOT-304: 386SX @ 16MHz, 2MB, VGA (vm/386sx)
make 386         # 86Box Micronics: 386DX @ 25MHz, 2MB, VGA (vm/386dx)
make xt-sound    # ...the XT again with a Sound Blaster 2.0 in it (vm/xt-sound)
make 286-sound   # 286 + SB16 (vm/286-sound)
make 386-sound   # 386DX + SB16 (vm/386-sound)
make check-images # are the git-tracked binaries in build/ what the sources build?
make clean
```

**`make check-images` before committing anything under `build/`.** `build/` is
gitignored, but ~21 artifacts inside it are force-added and shipped — the kernel,
both boot sectors, both bootable floppies, both software floppies, and every
package's `.bin`/`.o88`. Nothing makes them follow a source change, so they go
stale in silence: the tree still builds, still boots, and still looks right while
carrying a floppy image that no longer holds what the source says it does. That
is not hypothetical — two "Rebuild the shipped images" commits exist because
someone caught it by hand, and a merge shipped a Paint two fixes out of date
until the merge rebuilt it. The target builds everything a second time into
`build/.check` and compares byte for byte, which only works because the
toolchain is deterministic on purpose (`tools/os88disk.py` pins the volume
serial and every FAT timestamp for exactly this reason). It reads its list from
`git ls-files build`, so it cannot drift from what is actually tracked, and it
fails three ways: **STALE** (rebuild and commit), **ORPHAN** (tracked, nothing
builds it) and **SCRATCH** (a tracked `VIDEO=`/`RTC=` stamp — which has been
force-added twice, and which needs naming specially because two empty files
compare equal). Its comparison build is always knob-free, so a kernel built with
`VIDEO=`/`RTC=` that reached the tree reads as stale — which mechanizes the
warning the kernel recipe already prints.

Two build knobs exist only for testing the video fallbacks (SPEC.md §39.9):

```
make test VIDEO=cga                    # force the CGA path on a VGA machine
make test VIDEO=herc HERCSEG=0x7000    # force Hercules, framebuffer in RAM
python3 tools/hercshot.py build/qmp.sock 0x70000 out.png   # LINEAR = HERCSEG*16
python3 tools/mouse.py --screen 720x348 build/qmp.sock ...  # MANDATORY here
# ...the whole recipe, and the four ways to get it silently wrong, are in
# docs/HERCULES-TESTING.md
```

A third does the same for the clock (SPEC.md §37.90) — QEMU has an MC146818 and
nothing else, so the other three rungs of the RTC ladder are unreachable without it:

```
make test RTC=bios     # int 1Ah instead of the chip
make test RTC=none     # no clock at all: the 4 July 2026 fallback
make test RTC=ns       # the MM58167 probe against a machine that has none -
                       # it must REJECT and boot, not hang or invent a clock
```

`RTC=` shares `VIDEO=`'s stamp file, so changing it rebuilds the kernel; the
shipped images are always built without either.

`VIDEO=` is tracked by a stamp file, so changing it rebuilds the kernel — without that,
make sees an up-to-date `kernel.bin`, boots the previous adapter, and it reads exactly
like the probe being broken.

**Installing the toolchain in a fresh container (read this before fighting apt).**
`nasm` installs normally. `qemu-system-x86` does **not**: the package index

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jggonz/os8088](https://github.com/jggonz/os8088) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
