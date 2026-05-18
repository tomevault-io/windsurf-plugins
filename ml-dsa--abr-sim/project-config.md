---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

Pre-silicon trace generator for the [Adam's Bridge](https://github.com/chipsalliance/adams-bridge) PQC hardware accelerator (FIPS 204 ML-DSA / Dilithium and FIPS 203 ML-KEM / Kyber). It runs a full Verilator RTL simulation of an ML-DSA-87 or ML-KEM-1024 operation and emits VCD toggle traces suitable for TVLA-style side-channel leakage analysis.

## Baseline

The repo is on `adams-bridge` v2.0.3 (commit `b77e3d8`). The v2 migration is complete: `abr_top` replaced `mldsa_top`, the split `mldsa_seq_prim`/`mldsa_seq_sec` sequencers were collapsed into a single `abr_seq`, ML-KEM acquisition was added, and the new ML-DSA modes `mldsa-sign-extmu` and `mldsa-sign-stream` are wired up. `v2-plan.md` was the migration tracker and has been removed; `git show 4cdbb3a^:v2-plan.md` recovers the historical record if needed.

Known upstream limitation: there is no `MASKING_EN` build switch. In `v2.0.3`'s `abr_top`, masking is encoded in sequencer opcodes and internal control rather than exposed as a top-level parameter, so a masked-vs-unmasked build matrix would require an RTL-level patch rather than a parameter override.

## Prerequisites

- Verilator >= 5.037 (development version; older versions may not work)
- Standard C/C++ toolchain (`gcc`, `g++`)
- Python 3 with `pycryptodome` (provides `Crypto.Hash`) — used by `flow/fips204.py`, `flow/fips203.py`, `flow/mldsa-gen.py`, `flow/mlkem-gen.py`. Bash trace scripts honor `PYTHON=...` and default to `python3`.
- The `adams-bridge` git submodule must be checked out at the pinned release (currently v2.0.3, commit `b77e3d8`). After fresh clone: `git submodule update --init --recursive`.

## Build

```
make            # builds both readvcd and abr_wrap (Verilator step takes a minute or two)
make abr_wrap   # just the wrapped DUT binary
make lint-abr   # Verilator lint-only check
make clean      # also removes *.vcd, *.dat, _build/, _tr*/, plot/ artifacts
```

The Makefile build does three non-obvious things:

1. **Patches upstream RTL.** `rtl/abr_seq.sv` is *generated* by copying `adams-bridge/src/abr_top/rtl/abr_seq.sv` and applying `rtl/abr_seq.sv.patch`. The patch inserts a hook that `$display`s the current FSM state each cycle, decoded via `rtl/abr_seq_decode.sv` (covers both `MLDSA_*` and `MLKEM_*` symbols). The decoded output appears as `[seq]` lines in run output. If you bump the `adams-bridge` submodule, the patch will likely need to be regenerated.
2. **Verilator file list is generated** at `_build/xabr_wrap.vf` from upstream `adams-bridge/src/abr_top/config/abr_top.vf`. The Makefile materializes `${ADAMSBRIDGE_ROOT}` to the local submodule path, swaps in the patched `rtl/abr_seq.sv`, and appends `rtl/abr_seq_decode.sv` and `rtl/abr_wrap.sv`. Adding new local RTL means editing the Makefile's append step.
3. **Local wrapper.** `rtl/abr_wrap.sv` instantiates `abr_top` + `abr_mem_top` with `AHB_ADDR_WIDTH=32` and CALIPTRA disabled. This is the top-level module that `abr_wrap` (the C++ binary) drives.

## End-to-end trace flow

The pipeline has four stages, each producing inputs for the next:

1. **Vector generators.**
   - `flow/mldsa-gen.py <message> <xi> <rho'>` — pure-Python ML-DSA reference (`flow/fips204.py`) generates test vectors and writes `hash_in.dat`, `seed_in.dat`, `pk_in.dat`, `sk_in.dat`, `rnd_in.dat`, `mu_in.dat` (for external-µ), plus an "expected" `sig_in.dat` for cross-check. All args optional.
   - `flow/mlkem-gen.py {all|keygen|encaps|decaps} <d> <z> <m>` — pure-Python ML-KEM reference (`flow/fips203.py`) writes `seed_d_in.dat`, `seed_z_in.dat`, `msg_in.dat`, `ek_in.dat`, `dk_in.dat`, `ct_in.dat`, `ss_in.dat`.
2. **`./abr_wrap [options] <operation>`** — Verilated RTL sim with a descriptor-driven C++ AHB testbench (`src/abr_wrap.cpp`). Reads the `*_in.dat` files via the AHB register map (constants near the top of `abr_wrap.cpp`, mirrors `adams-bridge/src/abr_top/rtl/abr_reg.rdl`), runs the operation, writes the corresponding `*_out.dat`, and dumps a VCD trace (none by default). Operations:
   - ML-DSA: `mldsa-keygen`, `mldsa-sign`, `mldsa-verify`, `mldsa-kgsign`, `mldsa-sign-extmu` (caller-supplied µ), `mldsa-sign-stream` (variable-length byte stream via `MSG_STROBE`). The bare names `keygen` / `sign` / `verify` / `kgsign` are aliases for the `mldsa-*` versions.
   - ML-KEM: `mlkem-keygen`, `mlkem-encaps`, `mlkem-decaps`, `mlkem-kgdecaps`.
   After signing, a successful run satisfies `cmp sig_out.dat sig_in.dat`. Engine-specific status error masks live in the `Operation` table (`STATUS_MLDSA_ERROR=0x8`, `STATUS_MLKEM_ERROR=0x4`).
3. **`./readvcd <trace.vcd> <time-signal> [threshold] [filters/report-cycles]`** — single-pass C VCD parser. The `<time-signal>` is matched as a substring against signal names; the canonical choice (set in `flow/readvcd.prm`) is `dec.cyc` (the cycle counter inserted by `rtl/abr_seq_decode.sv`). Output is one `# <cycle> [togd] <count>` line per cycle. Optional hierarchy filters `-i <glob>` / `-e <glob>` restrict which VCD signals contribute to the toggle count; the timing signal is still used even if it is outside the selected hierarchy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-dsa/abr-sim](https://github.com/ml-dsa/abr-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
