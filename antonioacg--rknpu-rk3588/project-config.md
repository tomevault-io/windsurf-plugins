---
trigger: always_on
description: Out-of-tree RKNPU kernel module for RK3588/RK3588S on mainline Linux 6.19+. Enables the vendor RKNN SDK (librknnrt, rkllama, rknn-toolkit2) on modern kernels. The driver code exists in `ref/rknpu-module` (a git submodule of w568w/rknpu-module). Our job is the RK3588 device tree overlay, testing, and documentation.
---

# CLAUDE.md

## Project Purpose

Out-of-tree RKNPU kernel module for RK3588/RK3588S on mainline Linux 6.19+. Enables the vendor RKNN SDK (librknnrt, rkllama, rknn-toolkit2) on modern kernels. The driver code exists in `ref/rknpu-module` (a git submodule of w568w/rknpu-module). Our job is the RK3588 device tree overlay, testing, and documentation.

## Repository Layout

See [README.md § Repository Structure](README.md#repository-structure) for the canonical tree. Key paths:

- `dts/rk3588-rknpu-overlay.dts` — the main deliverable (DT overlay)
- `patches/*.patch` — carry changes applied to `ref/rknpu-module` at build time. Never edit the submodule itself.
- `tests/rknn_smoke.c` — minimal inference test used by `scripts/test-inference.sh`
- `scripts/{apply-overlay,build-module,install-dkms,test-inference,watch-npu}.sh` — the operational surface
- `ref/rknpu-module` — w568w's DKMS module, treated as read-only upstream
- `build/` — gitignored; `scripts/build-module.sh` stages the submodule here and applies patches. Safe to `rm -rf`.

## The Main Deliverable: `dts/rk3588-rknpu-overlay.dts`

This is the DT overlay that adds the NPU device tree node to a mainline kernel DTB. It must define:

### NPU node (`npu@fdab0000`)
- compatible: `"rockchip,rk3588-rknpu"`
- reg: `0xfdab0000` (core0), `0xfdac0000` (core1), `0xfdad0000` (core2) — each 0x10000
- interrupts: GIC_SPI 110, 111, 112 (IRQ_TYPE_LEVEL_HIGH)
- clocks: SCMI_CLK_NPU(6), ACLK_NPU0(287), ACLK_NPU1(276), ACLK_NPU2(278), HCLK_NPU0(288), HCLK_NPU1(277), HCLK_NPU2(279), PCLK_NPU_ROOT(291)
- resets: SRST_A_RKNN0(272), SRST_A_RKNN1(250), SRST_A_RKNN2(254), SRST_H_RKNN0(274), SRST_H_RKNN1(252), SRST_H_RKNN2(256)
- power-domains: RK3588_PD_NPUTOP(9), RK3588_PD_NPU1(10), RK3588_PD_NPU2(11)
- iommus: reference to rknpu_mmu
- status: "okay"

### IOMMU node (`iommu@fdab9000`)
- compatible: `"rockchip,iommu-v2"`
- reg: 0xfdab9000, 0xfdaba000, 0xfdaca000, 0xfdada000 — each 0x100
- interrupts: GIC_SPI 110, 111, 112
- status: "disabled" (IOMMU v2 has known bugs — test before enabling)

### OPP table
300, 400, 500, 600, 700, 800, 900, 1000 MHz with vendor voltage ranges.

### Phandle resolution
The overlay references `&cru`, `&scmi_clk`, `&power`. Check if the target DTB has `__symbols__`:
```bash
dtc -I dtb -O dts /path/to/rk3588s-orangepi-5-pro.dtb | grep -c __symbols__
```
If `__symbols__` exists: use standard `&label` references.
If not: must use hardcoded phandle values or rebuild DTB with `-@` flag.

## Key Differences: RK3566 vs RK3588

| Aspect | RK3566 (w568w working) | RK3588 (our target) |
|--------|----------------------|---------------------|
| Cores | 1 | 3 (independent power domains) |
| MMIO regions | 1 (`0xfde40000`) | 3 (`0xfdab0000`, `0xfdac0000`, `0xfdad0000`) |
| IRQs | 1 (SPI 151) | 3 (SPI 110, 111, 112) |
| Clocks | 4 | 8 |
| Resets | 2 | 6 |
| Power domains | 1 (`RK3568_PD_NPU`) | 3 (`NPUTOP`, `NPU1`, `NPU2`) |
| IOMMU reg regions | 1 | 4 |
| DMA mask | 32-bit | 40-bit |
| Max frequency | 800 MHz | 1000 MHz |
| SCMI clocks | Optional | Required (`SCMI_CLK_NPU`) |
| NPU GRF | Not needed | `npu_grf@fd5a2000` (syscon) |

## Reference Sources

All hardware details come from three authoritative sources:

1. **Rockchip vendor DTS** (`rk3588s.dtsi`): Defines the complete NPU node for the vendor driver. Primary reference. URL: `https://github.com/rockchip-linux/kernel/blob/develop-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi`
2. **Mainline kernel Rocket DTS** (`rk3588s.dtsi`): Same hardware, different driver — confirms addresses independently. Search mainline `torvalds/linux` for `rk3588-rknn-core`.
3. **Running hardware**: The Orange Pi 5 Pro already has the NPU active under kernel 6.1. Check `/proc/device-tree/npu@fdab0000/`, `/proc/interrupts`, `/sys/module/rknpu/`.

For clock/reset/power-domain constants:
- `include/dt-bindings/clock/rockchip,rk3588-cru.h` (mainline kernel)
- `include/dt-bindings/power/rk3588-power.h` (mainline kernel)
- `include/dt-bindings/reset/rockchip,rk3588-cru.h` (mainline kernel)

## Known Issues and Risks

1. **IOMMU v2 bug**: Mainline `rockchip-iommu.c` doesn't constrain page table allocations to DMA32 zone. If page tables land above 4GB, bus errors occur. Workaround: disable IOMMU in overlay. Test with IOMMU disabled first.

2. **Phandle resolution**: If the board DTB lacks `__symbols__`, the overlay can't resolve `&cru` etc. May need hardcoded phandle values or DTB rebuild.

3. **SCMI clocks**: RK3588 NPU clock (`SCMI_CLK_NPU`) is managed by the SCP via SCMI. The SCMI agent must be running. Verify with: `ls /sys/firmware/scmi_dev/*/`.

4. **Power domain sequencing**: RK3588 has per-core power domains (`NPUTOP`, `NPU1`, `NPU2`). The driver probes them in order and falls back to fewer cores if unavailable.

5. **40-bit DMA**: RK3588 uses 40-bit DMA addressing. If IOMMU is disabled and CMA is above 4GB, allocations may fail. Boot with `cma=128M` or `cma=64M@0-4G` as fallback.

## Submodule Grep Patterns

```bash
# Find all RK3588 NPU references across all submodules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonioacg/rknpu-rk3588](https://github.com/antonioacg/rknpu-rk3588) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
