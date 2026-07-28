---
trigger: always_on
description: OpenCCU (formerly RaspberryMatic) is a **Buildroot-based custom Linux operating system** that runs a cloud-free Homematic IP / HomeMatic smart-home hub. It aims for 100% CCU3 compatibility while adding OS-level enhancements.
---

# OpenCCU – Copilot Instructions

## Project Overview

OpenCCU (formerly RaspberryMatic) is a **Buildroot-based custom Linux operating system** that runs a cloud-free Homematic IP / HomeMatic smart-home hub. It aims for 100% CCU3 compatibility while adding OS-level enhancements.

Supported targets (14): `rpi3`, `rpi4`, `rpi5`, `tinkerboard2`, `odroid-c2`, `odroid-c4`, `odroid-n2`, `generic-aarch64`, `generic-x86_64`, `ova`, `lxc_amd64`, `lxc_arm64`, `oci_amd64`, `oci_arm64`.

## Build Commands

```bash
# Build a specific product (downloads Buildroot on first run)
make PRODUCT=rpi4 build

# Build all supported products
make build-all

# Create a release archive for one product
make PRODUCT=rpi4 release

# Run CI consistency checks for one product (requires Python flake8)
make PRODUCT=rpi4 check

# Run checks for all products
make check-all

# Interactive Buildroot config
make PRODUCT=rpi4 menuconfig

# Interactive Linux kernel config
make PRODUCT=rpi4 linux-menuconfig

# Save modified defconfig back to the repo
make PRODUCT=rpi4 savedefconfig
make PRODUCT=rpi4 linux-update-defconfig

# Clean build dir for one product
make PRODUCT=rpi4 clean

# Remove everything (all build dirs + downloaded Buildroot source)
make distclean
```

`make` without arguments prints the full list of available targets and supported products.

## Architecture

```text
buildroot-external/        # Buildroot BR2_EXTERNAL layer – all OpenCCU customization
  configs/                 # Per-product Buildroot defconfigs (e.g. rpi4.config)
  package/                 # Custom Buildroot packages (occu, rpi-rf-mod, generic_raw_uart, …)
  patches/occu/            # Upstream OCCU WebUI/firmware patches (see below)
  patches/<pkg>/           # Patches applied to other Buildroot packages
  board/<product>/         # Board-specific files: kernel defconfig, U-Boot config, DTS patches
  kernel/6.18/             # Shared kernel config fragments applied to all boards
  overlay/                 # Filesystem overlays merged into the target rootfs
  bootloader/              # U-Boot configuration
  scripts/                 # Helper scripts used during build
buildroot-patches/         # Patches applied to Buildroot itself before use
release/                   # Release scripts, EULA files, package manifests
home-assistant-addon/      # Published Home Assistant add-on
home-assistant-addon-dev/  # Development/pre-release add-on variant
helm/                      # Kubernetes Helm chart
.github/workflows/         # CI/CD: ci.yml, snapshot.yml, release.yml, …
```

The build system downloads `buildroot-2025.11.2.tar.gz`, applies `buildroot-patches/`, then invokes Buildroot with `BR2_EXTERNAL=buildroot-external`. Build output goes to `build-<product>/`.

## Key Conventions

### OCCU Patches

`buildroot-external/patches/occu/` holds all modifications to the upstream eQ-3 OCCU firmware. There are currently **322 patch directories** covering 635 individual files.

#### How Buildroot applies the patches

`buildroot-external/Buildroot.config` sets:

```make
BR2_GLOBAL_PATCH_DIR="$(BR2_EXTERNAL_EQ3_PATH)/patches"
```

Buildroot's global patch mechanism scans that directory for a subdirectory whose name matches the package being built (`occu`), then applies every `*.patch` file found there in **lexicographic / numeric sort order** during the `occu-patch` build step.

#### Directory layout

Every patch is represented by **two sibling items** sharing the same name — a directory and a `.patch` file:

```text
buildroot-external/patches/occu/
  0001-OpenCCU/              ← working-copy source tree for this patch
  0001-OpenCCU.patch         ← generated unified diff (applied by Buildroot)
  0002-WebUI-Bootstrap/
  0002-WebUI-Bootstrap.patch
  …
  0173-WebUI-SendPOSTRequest/
  0173-WebUI-SendPOSTRequest.patch
```

Inside each numbered directory the layout mirrors the upstream OCCU source tree under an `occu/` prefix. For every file that the patch modifies, **two files are kept side by side**:

```text
0001-OpenCCU/
  occu/
    WebUI/www/webui/webui.js        ← modified version (what goes into the build)
    WebUI/www/webui/webui.js.orig   ← upstream original (never edited)
    WebUI/www/config/cp_network.cgi
    WebUI/www/config/cp_network.cgi.orig
    …
```

The `.orig` file is **always the verbatim upstream content** for that OCCU version. The file without `.orig` is the OpenCCU-modified version. Diffs are generated from `.orig` → no-extension.

#### create_patches.sh — what it does

`buildroot-external/patches/occu/create_patches.sh` regenerates **all** `.patch` files from the working-copy source pairs:

1. Deletes every existing `*.patch` file in the directory.
2. Iterates numbered directories in sorted order.
3. For each directory, finds all `*.orig` files under `occu/`.
4. For each `*.orig` file, runs:
   ```sh
   diff -u --label="${file}" --label="${file%.orig}" "${file}" "${file%.orig}" >> "../${dir}.patch"
   ```
   The `--label` flags produce standard `---`/`+++` headers with the `.orig` suffix stripped from the destination label, matching what Buildroot's `patch` command expects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenCCU/OpenCCU](https://github.com/OpenCCU/OpenCCU) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
