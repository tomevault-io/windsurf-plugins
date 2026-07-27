---
trigger: always_on
description: **Calculinux** is a Yocto-based Linux distribution for the **ClockworkPi Picocalc** device (currently supporting Luckfox Lyra SBC). Key architectural features:
---

# Copilot Instructions for Calculinux

## Project Overview

**Calculinux** is a Yocto-based Linux distribution for the **ClockworkPi Picocalc** device (currently supporting Luckfox Lyra SBC). Key architectural features:

- **KAS-based Build System**: Uses KAS for reproducible configuration management (`kas-base.yaml` + `kas-luckfox-lyra-bundle.yaml`)
- **Multi-layer Architecture**: 
  - `meta-calculinux-distro`: Core distribution config, system image, RAUC update bundles
  - `meta-picocalc-bsp-rockchip`: Board support (kernel, u-boot, device trees, WIC partitioning)
  - `meta-calculinux-apps`: Application recipes (x48ng, kiwix-tools, SDL test apps)
  - `meta-meshtastic`: Meshtastic connectivity support
- **Read-only Root + OverlayFS**: Root filesystem on MMC is read-only; user data persists via OverlayFS on SD card
- **RAUC A/B Updates**: Dual rootfs partitions (ROOT_A/ROOT_B) with atomic OTA updates using dm-verity
- **Device Tree ConfigFS**: Runtime overlay loading via in-kernel `drivers/of/configfs` (no external module)
- **Custom Hardware Drivers**: Picocalc keyboard, LCD (ILI9488 DRM), PWM audio, MFD (from `Calculinux/picocalc-drivers` repo)

## Build System Workflow

### CRITICAL Build Location Rules
- **NEVER** run build commands from within `meta-calculinux/` directory - creates unwanted build artifacts
- **ALWAYS** run from parent `calculinux-build/` directory (typically `/home/<username>/repos/calculinux/calculinux-build/`)
- The `./build` symlink in `meta-calculinux/` points to parent directory but prefer explicit paths

### KAS Container Commands (from calculinux-build/)
```bash
# Full image build
./meta-calculinux/kas-container build ./meta-calculinux/kas-luckfox-lyra-bundle.yaml

# Interactive shell for bitbake commands
./meta-calculinux/kas-container shell ./meta-calculinux/kas-luckfox-lyra-bundle.yaml

# Build specific recipe in shell
./meta-calculinux/kas-container shell ./meta-calculinux/kas-luckfox-lyra-bundle.yaml -c "bitbake <recipe-name>"
```

### Build Artifacts Locations
**IMPORTANT**: Build directory is `../build/` relative to `meta-calculinux/` (i.e., `/home/<username>/repos/calculinux/calculinux-build/build/`), NOT `/build/`

- **Images**: `../build/tmp/deploy/images/luckfox-lyra/calculinux-image-luckfox-lyra.rootfs.wic`
- **RAUC bundles**: `../build/tmp/deploy/images/luckfox-lyra/calculinux-bundle-luckfox-lyra.raucb`
- **IPK packages**: `../build/tmp/deploy/ipk/` (organized by architecture: armv7ahf-neon-vfpv4, all, luckfox_lyra)
- **Build work directory**: `../build/tmp/work/<arch>/<recipe>/<version>/` (contains source, build logs, staging)
- **Compile logs**: `../build/tmp/work/<arch>/<recipe>/<version>/temp/log.do_compile.*`

### Build Targets
- `calculinux-image`: Main system image (defined in `meta-calculinux-distro/recipes-core/image/calculinux-image.bb`)
- `calculinux-bundle`: RAUC update bundle (defined in `meta-calculinux-distro/recipes-core/bundles/calculinux-bundle.bb`)
- `packagegroup-meta-calculinux-apps`: Application bundle (apps layer)

**Always wait for builds to complete before declaring success/failure.**

## Terminal Command Output Guidelines

### IMPORTANT: Do NOT Pipe Long-Running Commands to head/tail
- **NEVER** use `| head`, `| tail`, or similar output limiting for long-running build/fetch tasks
- This removes all intermediate output, hiding progress and diagnostics
- Users cannot see what's happening or debug issues when the command fails
- Run commands without piping to capture full output
- Exception: Only use output limiting when explicitly inspecting specific data (e.g., `grep` results)

**Example - WRONG:**
```bash
./meta-calculinux/kas-container build ... | tail -20  # BAD: hides build progress
```

**Example - CORRECT:**
```bash
./meta-calculinux/kas-container build ...  # GOOD: full output visible
```

## Patch Creation Guidelines - READ CAREFULLY

### ABSOLUTELY CRITICAL
**DO NOT** attempt to create or edit patches by hand - spacing/whitespace errors make patches invalid.

### Required Process for ALL Patches
1. Fetch or checkout the actual source code to be patched
2. Make a working copy of the original file
3. Apply your changes to the copy
4. Generate patch using `diff -Naur` or `git diff` against actual modified code
5. **NEVER** fabricate patch content synthetically

### When Modifying Existing Patches
- Retrieve sources, apply patch, modify sources, regenerate full patch from modified sources
- You MAY edit: file paths, patch header comments, remove unwanted hunks
- You MAY NOT: hand-edit patch hunks, "fix" whitespace in patches, adjust line numbers manually

### Patch Application in Recipes
- Patches referenced via `SRC_URI` in KAS files: Use `patches:` section (see `kas-luckfox-lyra-bundle.yaml` for examples)
- Patches in `.bbappend` files: Add to `SRC_URI:append` 
- Store patches in `patches/` directory or recipe-local `files/` subdirectory

## Recipe Development

### Yocto Version and Modern Syntax

**Yocto Version**: Calculinux is based on **Yocto Scarthgap (2024.x)** via Poky

**Important Syntax Changes** - Avoid deprecated patterns:

| ❌ AVOID | ✅ USE INSTEAD | REASON |
|---------|----------------|--------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Calculinux/meta-calculinux](https://github.com/Calculinux/meta-calculinux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
