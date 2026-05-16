---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HighTide is a VLSI design benchmark suite built on OpenROAD-flow-scripts (ORFS). It ports open-source hardware designs to asap7/sky130hd/nangate45 technologies using the OpenROAD RTL-to-GDSII flow, serving as a benchmark suite for ML projects.

## Setup

Requires an Ubuntu machine with [Bazelisk](https://github.com/bazelbuild/bazelisk) (or Bazel 7.6.1+). No additional setup needed — Bazel fetches everything via `MODULE.bazel` and the pinned `bazel-orfs` submodule.

## Build Commands

```bash
# Build the full RTL-to-GDS flow for one design
bazel build //designs/asap7/lfsr:lfsr_final

# Build a design across all available platforms
bazel build //designs/asap7/minimax:minimax_final \
            //designs/nangate45/minimax:minimax_final \
            //designs/sky130hd/minimax:minimax_final

# Build all designs for a platform
bazel build //designs/asap7/...

# Build all designs across all platforms
bazel build //designs/...

# Build with dev RTL generation (requires submodule init + tools)
bazel build --define update_rtl=true //designs/asap7/lfsr:lfsr_final

# Build individual stages (target suffixes: _synth, _floorplan, _place, _cts, _grt, _route, _final)
bazel build //designs/asap7/lfsr:lfsr_synth
bazel build //designs/asap7/lfsr:lfsr_place
```

Note: `hightide_design()` exposes only the per-stage `orfs_flow` targets (`_synth` … `_final`) plus `_generate_abstract` / `_generate_metadata`. There is no aggregate `:<design>` target — use `:<design>_final` for the full flow.

## Architecture

### Key Relationships

- **`MODULE.bazel`** — Declares dependencies on `bazel-orfs` (via `git_override`) and configures the ORFS Docker image for tool extraction
- **`defs.bzl`** — `hightide_design()` macro wrapping `orfs_flow()` with common defaults (`GDS_ALLOW_EMPTY`, platform-to-PDK mapping)
- **`BUILD.bazel`** (root) — Defines `//:update_rtl` config setting and the `merge_yosys_share` target that bundles the yosys-slang plugin
- Each design has a `BUILD.bazel` calling `hightide_design()` with its parameters (utilization, density, SRAMs, etc.)
- RTL sources at `designs/src/<design>/BUILD.bazel` use `select()` to switch between release and dev RTL

### Design Configuration

Each design lives at `designs/<platform>/<design>/` and contains:
- **`BUILD.bazel`** — Calls `hightide_design()` with verilog_files, sources (SDC, LEFs, LIBs), arguments (utilization, density, etc.)
- **`constraint.sdc`** — Clock definitions and timing constraints
- **`pdn.tcl`** (optional) — Power delivery network configuration (layer stripes, pitches)
- **`io.tcl`** (optional) — Pin placement constraints

### RTL Source Management (`designs/src/`)

Design sources are git submodules under `designs/src/<design>/dev/repo/`.

Each `designs/src/<design>/BUILD.bazel` defines:
- `rtl_release` filegroup — pre-generated Verilog files
- `rtl_dev_gen` genrule — runs setup.sh/sv2v to generate from source
- `rtl` alias — uses `select({"//:update_rtl": ..., "//conditions:default": ...})` to pick

Dev mode requires: `git submodule update --init designs/src/<design>/dev/repo` before `bazel build --define update_rtl=true`.

### Platforms

| Platform | Node | Designs |
|----------|------|---------|
| asap7 | 7nm academic | coralnpu, gemmini, lfsr, litedram, minimax, sha3, vortex, liteeth_udp_usp_gth_sgmii, NVDLA (partitions a/c/m/o/p), bp_processor (bp_uno, bp_quad), cnn, floonoc, NyuziProcessor, snitch_cluster |
| nangate45 | 45nm | coralnpu, gemmini, lfsr, litedram, minimax, NyuziProcessor, sha3, liteeth_udp_usp_gth_sgmii, bp_processor (bp_uno, bp_quad), cnn |
| sky130hd | 130nm open | gemmini, lfsr, litedram, minimax, sha3, liteeth_udp_usp_gth_sgmii, cnn |

#### Build status (as of 2026-05-11)

Designs reaching `_final` (cached on remote build cache):
- **asap7**: coralnpu, gemmini, lfsr, litedram, minimax, sha3, vortex, liteeth_udp_usp_gth_sgmii, NVDLA partitions a/m/o
- **nangate45**: coralnpu, gemmini, lfsr, litedram, minimax, NyuziProcessor, sha3, liteeth_udp_usp_gth_sgmii
- **sky130hd**: gemmini, lfsr, litedram, minimax, sha3, liteeth_udp_usp_gth_sgmii, NVDLA partitions a/m/o/p

Not yet finishing (not cached):
- **asap7**: cnn, floonoc, NyuziProcessor, snitch_cluster, bp_processor (bp_uno, bp_quad), NVDLA partitions c, p
- **nangate45**: cnn, bp_processor (bp_uno, bp_quad)
- **sky130hd**: cnn, NVDLA partition c

`sky130hd/NVDLA/partition_c` global placement plateaus at overflow ~0.31 (target 0.10) — 84 macros at sky130hd's coarse pitches create local density hot spots near macro pin clusters that the placer can't smooth out. Loosening `CORE_UTILIZATION` / `PLACE_DENSITY_LB_ADDON` / `MACRO_PLACE_HALO` to match working partitions doesn't fix it. Likely needs a manual `macros.tcl` to spread the 84 SRAMs into a regular grid.

Use `tools/fetch_cache.sh` to pull cached `_final` results from the remote cache; designs marked NOT CACHED there are the not-yet-finishing set above.

### Output Directories

Build artifacts go to `bazel-bin/designs/<platform>/<design>/`. Key outputs:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VLSIDA/HighTide](https://github.com/VLSIDA/HighTide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
