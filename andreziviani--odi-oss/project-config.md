---
trigger: always_on
description: Guidance for any coding agent (or human) working in this repository. Read
---

# AGENTS.md

Guidance for any coding agent (or human) working in this repository. Read
the top-level `README.md` first for what this project is; this file is about
how to work in it safely and correctly. `docs/HACKING.md` has the detail
behind every rule here.

## Layout

    Makefile                the whole build; `make help` lists every target
    toolchain/               which toolchain images we build with: images.env (digest pins), image.sh; built in the odi-toolchain repo
    kernel/                  Linux 6.18 for the RTL9602C
      618/fetch.sh, mainline/, patches/, config
      618/debug/             debug-only patches, applied with CRUMBS_CORE=1
      build.sh
      extra/                 every file of our own, at its tree path (board, CPU cache, irqchip, timer, SPI NOR)
      extra/drivers/net/ethernet/odi/   our own switch/GPON/NIC/OMCI/watchdog/ramlog drivers
    packages/                upstream userland: busybox, dropbear, iproute2 (bridge-utils deliberately not built)
    rootfs/skeleton/         /etc and the init chain, ours; lib/firmware/odi/ the register replay tables
    image/                   assembles squashfs + uImage into a flashable tarball, and the on-device flasher (fwu.sh)
    src/                     our own tools: diag, omci (omcid/omcli/omciprobe/omcicap), igmp, nv — freestanding
    test/                    host-side and qemu-based test harnesses
    tools/                   developer tooling: remote-build.sh, kernel-footprint.sh, objdiff.sh, fetch-refs.sh, memprobe, regdump, regtrace, lint helpers
    docs/                    HACKING.md (contributor guide), BOOT.md, KERNEL.md, SWITCH.md, TOOLS.md, SETTINGS.md, BUILDING.md, FLASHING.md, ACCESS.md, CROSS-COMPILING.md, IMPROVEMENTS.md, LICENSING.md, REFERENCES.md; kb/ field notes

## Build and test commands

    make image-all           # everything, from a clean clone
    make image                # just the tarball, once the pieces exist
    make test                  # lint + test-host + test-diag + test-omci (~2 min)
    make test-host              # host-side unit tests, no Docker/stick needed
    make test-diag              # diag, including the exporter contract (byte-for-byte goldens)
    make test-rcs               # what rcS executes and writes to /proc, three flag sets, against goldens (needs out/busybox)
    make lint                   # shellcheck + repo-specific checks, see below

    kernel/build.sh              # the kernel alone (pulls the toolchain image itself)
    ODI_REMOTE=user@yourbuildhost tools/remote-build.sh 'make image-all'
                                  # build on a native x86_64 Linux host instead of
                                  # under emulation; see docs/BUILDING.md. ODI_REMOTE
                                  # is a placeholder you must set — this repo names
                                  # no host of its own.

`docs/BUILDING.md` has the full breakdown of every build target.

## House rules

- **No proprietary or vendor-sourced code or references, anywhere in this
  repo.** This project builds a complete replacement firmware from mainline
  Linux, public specifications (ITU-T G.984.3, G.988, SFF-8472) and our own
  independent implementation, written from hardware behaviour observed on
  the device; it ships and links no proprietary code. Do not
  add anything derived from, or copied from, a vendor's SDK or firmware
  source, and do not name proprietary vendor files, functions or symbols in
  code, comments, or docs — describing how the **stock (OEM) firmware
  behaves**, as an observed black box, is fine and often necessary; reading
  or citing its source is not.
- **No apostrophes in shell-script comments.** A single quote inside a
  single-quoted inline block (several scripts here run `docker run ... bash
  -c '...'`, where the whole block is one shell word) silently terminates
  the word and runs the rest of the line in the outer shell — this has cost
  real debugging time more than once. Rephrase; do not escape.
  `tools/check-inline-quotes.sh` (run by `make lint`) enforces this inside
  inline `bash -c` blocks specifically, but the safest habit is to avoid
  apostrophes in shell comments everywhere.
- **Every user-visible improvement goes into the README comparison table**
  (`README.md`, "area / stock / this firmware") in the same change, with
  a number where there is one, and only claims you can back: a
  measurement, a test, or the code.
- **Keep the mainline footprint minimal.** `kernel/618/patches/` holds only
  edits to files that exist in mainline; every new file lives in the
  `kernel/extra` overlay. Before adding a hunk to a mainline file, look for
  a hook that already exists (mach headers, board callbacks, our own Kconfig
  `select`s). Fold a fix into the patch it belongs to rather than stacking
  a new one, so each patch stays one coherent change, and check
  `tools/kernel-footprint.sh` before and after: the count only goes down
  without a stated reason. This is what makes the next LTS port cheap.
- **Name the test sticks by their line, ISP1 and ISP2**, in code, comments,
  docs and commit messages, and never write a credential, a serial number
  or another identity value from a real stick into the tree. `make lint`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreZiviani/odi-oss](https://github.com/AndreZiviani/odi-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
