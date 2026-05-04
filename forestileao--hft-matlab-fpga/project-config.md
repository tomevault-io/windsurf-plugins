---
trigger: always_on
description: This repository is a research prototype for a low-cost FPGA-assisted HFT pipeline. The current codebase centers on a shared-memory streaming bridge between ARM software and FPGA logic, plus a small FAST market-data feed generator/receiver pair used as a software-side harness.
---

# AGENTS.md

## Purpose

This repository is a research prototype for a low-cost FPGA-assisted HFT pipeline. The current codebase centers on a shared-memory streaming bridge between ARM software and FPGA logic, plus a small FAST market-data feed generator/receiver pair used as a software-side harness.

If you are making changes here, optimize for preserving the contract between:

- `cpp/src/fpga_shared_stream.h`
- `cpp/src/fast_receiver.cpp`
- `vhdl/arm_fpga_shared_stream_bridge.vhd`
- `docs/arm-fpga-shared-memory-stream.md`

Those four files define the most important integration surface in the repo.

## Repository Map

- `README.md`: project context, goals, and top-level test commands.
- `Makefile`: primary entrypoint for builds and tests. Uses Docker for all supported workflows.
- `Dockerfile`: Ubuntu 24.04 image with CMake, Boost, Git, and GHDL.
- `cpp/`: C++ utilities and tests.
- `cpp/src/templates/SimpleMD.xml`: mFAST template used to generate message types.
- `cpp/src/fast_data_feed.cpp`: TCP server that emits encoded FAST messages.
- `cpp/src/fast_receiver.cpp`: TCP client that decodes FAST messages and optionally forwards them into the FPGA MMIO bridge.
- `cpp/src/fpga_shared_stream.h`: C++ wrapper for the ARM/FPGA shared-memory ring interface.
- `cpp/tests/fpga_shared_stream_test.cpp`: host-side unit test for the MMIO wrapper using a temporary backing file.
- `vhdl/arm_fpga_shared_stream_bridge.vhd`: shared-memory bridge RTL.
- `vhdl/tb_arm_fpga_shared_stream_bridge.vhd`: basic functional testbench.
- `vhdl/tb_arm_fpga_shared_stream_bridge_fast.vhd`: burst, wrap-around, and backpressure testbench.
- `docs/arm-fpga-shared-memory-stream.md`: protocol/register-map documentation for the bridge.
- `matlab/hft.prj`: MATLAB HDL Coder project metadata. At the moment this is mostly configuration, not a full MATLAB algorithm source tree.

## How The Repo Actually Works

The implemented prototype is not yet a full trading system. The concrete, working path today is:

1. `fast_data_feed` publishes synthetic FAST messages over TCP on port `9001`.
2. `fast_receiver` connects to that feed, decodes messages with mFAST, and prints them.
3. If `HFT_FPGA_MMIO_BASE` is set, `fast_receiver` also packs each decoded entry into a 128-bit frame and writes it into the ARM-to-FPGA TX ring.
4. The VHDL bridge exposes two ring buffers via MMIO:
   - TX ring: ARM -> FPGA
   - RX ring: FPGA -> ARM
5. Responses can be written by FPGA logic into the RX ring and read back by software.

## Build And Test Workflow

Use the `Makefile`. The intended flow is Dockerized so host toolchain differences do not matter.

Common commands:

- `make docker-image`
- `make cpp-test`
- `make vhdl-test`
- `make vhdl-test-fast`
- `make vhdl-wave`
- `make docker-shell`

C++ builds depend on a local checkout/install of `mFAST`, which the Make targets manage automatically through:

- `make mfast-clone`
- `make mfast-install`
- `make cpp-configure`
- `make cpp-build`

Notes:

- `cpp/CMakeLists.txt` auto-detects `../mFAST/install` if present.
- C++ tests build only `fpga_shared_stream_test`; the FAST sender/receiver are build targets, not covered by automated tests here.
- VHDL simulations generate VCDs under `vhdl/build/`.

## Runtime Knobs

`fast_receiver` enables the FPGA bridge only when these environment variables are provided:

- `HFT_FPGA_MMIO_BASE`: required physical base address
- `HFT_FPGA_MMIO_SPAN`: optional MMIO span, defaults to `0x1000`
- `HFT_FPGA_MMIO_DEV`: optional device path, defaults to `/dev/mem`

If `HFT_FPGA_MMIO_BASE` is unset, the receiver stays in software-only mode and just prints decoded messages.

## Cross-Language Invariants

When editing the shared stream interface, keep these aligned across C++, VHDL, tests, and docs:

- register offsets
- queue ownership rules
- ring full/empty semantics
- slot width in 32-bit words
- frame word ordering
- status bit meanings

Current frame packing from `fast_receiver.cpp` is:

- `word0`: sequence number
- `word1`: first 3 symbol bytes plus side code in the top byte
- `word2`: price scaled by `1e4`
- `word3`: quantity

The VHDL bridge stores slot lanes little-endian by 32-bit word index, and the testbenches assert that exact ordering.

## Important Pitfalls

### 1. RX base is conceptually dynamic

The HDL and docs define:

- `TX_BASE = 0x100`
- `RX_BASE = TX_BASE + DEPTH * SLOT_WORDS * 4`

`cpp/src/fpga_shared_stream.h` currently hardcodes:

- `kTxBase = 0x100`
- `kRxBase = 0x500`

That hardcoded `0x500` is correct only for the default layout (`DEPTH=64`, `SLOT_WORDS=4`). If you make queue geometry configurable in software, compute RX base dynamically from the discovered register values.

### 2. Ring capacity is `DEPTH - 1`

Both software and HDL use the standard single-producer/single-consumer convention where `next(head) == tail` means full. Do not accidentally treat all `DEPTH` slots as usable.

### 3. Publish order matters


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forestileao/hft-matlab-fpga](https://github.com/forestileao/hft-matlab-fpga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
