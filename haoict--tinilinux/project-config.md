---
trigger: always_on
description: These notes make AI agents productive quickly in this Buildroot-based distro. Focus on the actual patterns in this repo — not generic advice.
---

# Copilot Instructions for TiniLinux

These notes make AI agents productive quickly in this Buildroot-based distro. Focus on the actual patterns in this repo — not generic advice.

**Big Picture**
- **Buildroot External Tree:** This repo is a Buildroot BR2_EXTERNAL that defines boards, overlays, and custom packages. Build output lives in `output/<board>` created via an out-of-tree build alongside a sibling `buildroot/` directory. See [external.desc](../external.desc) and [external.mk](../external.mk).
- **Boards as Variants:** Each board name maps 1:1 to a defconfig in [configs/](../configs). Matching board directories live under [board/](../board) for BOOT, rootfs overlays, and board-specific assets. Board variants (e.g., `h700`, `h700_sway`, `h700_rootrw`, `h700_consoleonly`) share configs via fragments.
- **Custom Packages:** All packages reside in [package/](../package) and are auto-included via `include $(wildcard $(BR2_EXTERNAL_TiniLinux_PATH)/package/*/*.mk)` in [external.mk](../external.mk). A top-level [Config.in](../Config.in) exposes package menus grouped by function: "TiniLinux Common Packages" (btop, gptokeyb2, initramfs, etc.), "TiniLinux Graphic Packages" (mesa3d-no-llvm, retroarch, simple-launcher, etc.), and "TiniLinux RK3566 Packages" (rk3566-dtbo).
- **Init System + Kernel:** Systemd-based images with board-specific kernels and U-Boot patches defined in each `*_defconfig`. Example: [h700_sway_defconfig](../configs/h700_sway_defconfig).
- **Architecture:** Targets embedded ARM64 devices (Rockchip RK3326/RK3566, Allwinner H700) with GPU acceleration via Panfrost Mesa driver. Also supports QEMU virtual boards (`qemu_aarch64`) and Raspberry Pi 3B.

**Repo Layout**
- **Boards:** [board/h700](../board/h700), [board/rgb30](../board/rgb30), [board/qemu_aarch64](../board/qemu_aarch64), [board/pi3b](../board/pi3b) plus  `_sway`, `_consoleonly`, `_development` variants. Default configs (h700, rgb30) use squashfs rootfs. Each board dir contains:
  - `BOOT/` - bootloader assets, device trees, extlinux config
  - `rootfs/` - overlay files for ext4 rootrw variants
  - `overlay_upper/` - overlay files for squashfs variants (persistent overlay partition)
  - `ROMS/` - optional RetroArch configs and cores (mainly in `board/common/ROMS`)
- **Configs:** [configs/](../configs) holds all `<board>_defconfig` and toolchain-only defconfigs. Most defconfigs use fragments via `BR2_DEFCONFIG_FRAGMENT` to reduce duplication. Example: `h700_defconfig` is just 2 lines referencing fragments and overlay paths.
- **Config Fragments:** [configs/fragments/](../configs/fragments) contains reusable config fragments: `common.fragment` (shared by all), `h700.fragment`/`rgb30.fragment`/`pi.fragment` (board-specific), `with-graphics.fragment` (GUI packages), `rootrw.fragment`, `sway.fragment`, `qemu.fragment`.
- **Packages:** Examples: [package/initramfs](../package/initramfs), [package/simple-launcher](../package/simple-launcher), [package/mesa3d-no-llvm](../package/mesa3d-no-llvm), [package/rk3566-dtbo](../package/rk3566-dtbo). Each package has `<name>.mk` (Makefile) and `Config.in` (menu entry).
- **Tooling:** [make-board-build.sh](../scripts/make-board-build.sh) bootstraps an out-of-tree Buildroot output, auto-clones buildroot if needed, and merges fragments; [Dockerfile](../Dockerfile) provides Ubuntu 24.04 build container with all deps.
- **CI/CD:** [.github/workflows/build.yaml](../.github/workflows/build.yaml) defines manual workflow_dispatch builds with caching for `dl/` (downloads) and `.buildroot-ccache/` (compiled objects). Supports multiple runner types including GitHub-hosted and self-hosted ARM runners.
- **Docs:** Start with [README.md](../README.md). Board-specific notes may exist in `board/<board>/README`.

**Build Workflow**
- **Prerequisites:** Build environment requires `build-essential cmake mtools libncurses-dev dosfstools parted`. The buildroot repo is auto-cloned by [make-board-build.sh](../scripts/make-board-build.sh) if not present as a sibling `../buildroot/`.
- **Directory structure:** Expected layout is `TiniLinux/` (this repo) and `buildroot/` (auto-cloned) as siblings, with build outputs in `TiniLinux/output/<board>` or `buildroot/output/<board>` (for Docker builds).
- **Bootstrap build dir:**
  - `./scripts/make-board-build.sh configs/<board>_defconfig` → creates `output/<board>`, merges fragments if used, and wires `BR2_EXTERNAL`. Pass `docker` as second arg to adjust paths to use native docker volume.
- **Configure and build:**
  - `cd output/<board>` → `make menuconfig` (optional) → `make -j$(nproc)`.
- **Save config changes:**
  - `make savefconf` → saves minimal config while preserving `BR2_DEFCONFIG_FRAGMENT` structure. Use this instead of `make savedefconfig` for fragment-based configs. Implemented in [save-fragment-defconfig.sh](../scripts/save-fragment-defconfig.sh).
- **Image creation:**
  - `make img` invokes [external.mk](../external.mk) which selects either [mk-flashable-img-rootrw-rootless.sh](../scripts/mkimg/mk-flashable-img-rootrw-rootless.sh) or [mk-flashable-img-squashfs-rootless.sh](../scripts/mkimg/mk-flashable-img-squashfs-rootless.sh) based on presence of `root.img`.
- **Flash to SD:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haoict/TiniLinux](https://github.com/haoict/TiniLinux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
