---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

OpenIPC **builder** is a thin Buildroot *overlay* layer for building OpenIPC IP-camera
firmware for specific, named consumer devices. It contains **only the per-device deltas**
(a defconfig, a first-boot customizer, a rootfs exclude list, occasional sensor/board
files). Everything common — the actual Buildroot tree, packages, kernels, toolchains — lives
in [`OpenIPC/firmware`](https://github.com/openipc/firmware), which `builder.sh` clones fresh
on every run. This repo is never built in isolation; it is layered on top of a firmware
checkout.

## Build commands

```sh
./builder.sh                       # interactive whiptail menu of all devices
./builder.sh <device>              # build one device non-interactively
```

`<device>` is the directory name under `devices/` (minus the `_defconfig` suffix on its
config), e.g. `hi3518ev200_lite_switcam-hs303`. It is passed straight through as
`make BOARD=<device>`. There is no separate test/lint suite — "passing" means the firmware
image builds and (ideally) boots on hardware.

What `builder.sh <device>` does, in order:
1. `git pull` (self-update the builder repo).
2. `rm -rf openipc` then clone `OpenIPC/firmware` — HEAD by default, or the ref in
   `$OPENIPC_FW_REV` if set (used for cross-repo bisects; see build-one.yml).
3. `copy_extra_packages` — copy `package/*` into `openipc/general/package/` and append a
   `source "...Config.in"` line for each into the external tree's `Config.in`.
4. Copy `devices/<device>/*` over the firmware tree (defconfig, overlay, excludes, board).
5. `make BOARD=<device>` then best-effort `make BOARD=<device> size-report`.
6. `copy_to_archive` → `archive/<device>/<timestamp>/`. For `hi3518ev200_lite` it also runs
   `autoup_rootfs` to wrap the images as `autoupdate-*.img` via `mkimage`.

`openipc/`, `archive/`, `cache/`, `output/` are all gitignored build artifacts.

### Other scripts
- `repack.sh [uboot] [firmware] [ssid] [pass]` — does **not** build. Downloads a prebuilt
  release `.tgz` + u-boot from GitHub releases, optionally bakes in WiFi creds via
  `fw_setenv`, and `dd`s a flashable NOR image. Needs `squashfs-tools`.
- `package.sh [pkg]` — force a full rebuild of one Buildroot package inside an existing
  `openipc/` tree (`dirclean` + `rebuild`; defaults to `busybox`).

## Device anatomy

Device directory names encode `<soc>_<flavor>_<vendor>-<model>[-<version>]`:
- **soc** — OpenIPC SoC name: `hi3518ev200`, `ssc337de`, `t31`, `gk7205v200`, …
- **flavor** — firmware track: `lite` (default, the vast majority), `ultimate`, `fpv`,
  `rubyfpv`, `apfpv`. Prefer `lite` for new devices unless flash size forces otherwise.

The minimal required files for a registered device (per README "Requirements"):
```
devices/<device>/br-ext-chip-<vendor>/configs/<device>_defconfig
devices/<device>/general/overlay/usr/share/openipc/customizer.sh
devices/<device>/general/scripts/excludes/<soc>_<flavor>.list
```
- **`br-ext-chip-<vendor>/`** — the chip-vendor folder mirrors firmware's `BR2_EXTERNAL`
  layout. One of: `br-ext-chip-hisilicon`, `br-ext-chip-sigmastar`, `br-ext-chip-goke`,
  `br-ext-chip-ingenic` (Ingenic = the T-series SoCs). The defconfig inside selects
  toolchain/kernel/SoC drivers and majestic/webui packages; it references `$(OPENIPC_*)`
  and `$(EXTERNAL_VENDOR)` macros resolved by the firmware Makefile, not by this repo.
- **`customizer.sh`** — runs on first boot. Applies device-specific runtime config:
  `fw_setenv` (upgrade URL, `wlandev`, `ptz` profile, WiFi) and `cli -s .<path> <value>`
  to seed majestic's config (sensor ini, IR-cut/backlight GPIO pins, codec, fps).
- **`<soc>_<flavor>.list`** — paths to **delete** from the rootfs (unused sensor `.so`/`.ini`,
  unused WiFi `.ko`, etc.) so the image fits NOR flash. This is the main lever for the
  "doesn't fit in 8M NOR" problem.

Optional per-device files: extra `general/overlay/...` payload (a sensor `libsns_*.so`, a
sensor `.ini`, a patched `load_hisilicon`), or a custom kernel config at
`br-ext-chip-<vendor>/board/<family>/<soc>.generic.config`.

### `devices/common/`
Generic, non-device-specific defconfigs (`*_fpv`, `*_venc`, `*_lte`, `*_mini`) plus their
shared kernel configs and exclude lists. These are the matrix entries with a single
underscore (`hi3516ev200_fpv`, `hi3518ev200_mini`). The CI artifact-naming logic keys off
this: `COMMON = (underscore count) - 1`; when `COMMON == 1` the firmware's canonically-named
image is uploaded as-is, otherwise the compound `<soc>_<flavor>_<vendor>-<model>` image is
renamed to `<device>-nor.tgz` / `-nand.tgz` to avoid release-asset collisions.

## Adding a new device

The fastest correct path is to clone the closest existing device — **same SoC and same
flavor** — and edit the deltas. Identify the target's SoC, image sensor, WiFi chip, and flash
size first (the README device table lists all of these for existing boards).

1. **Copy a sibling.** `cp -r devices/<soc>_<flavor>_<other> devices/<soc>_<flavor>_<vendor>-<model>`.
   The new directory name *is* the `BOARD`/`<device>` token; keep it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenIPC/builder](https://github.com/OpenIPC/builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
