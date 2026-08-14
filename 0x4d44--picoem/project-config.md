---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## picoem — RP2350 / RP2040 Emulator Workspace

Cycle-accurate emulators for the Raspberry Pi RP2354 / RP2350 family (dual Cortex-M33 + PIO) and the RP2040 (dual Cortex-M0+ + PIO). Rust workspace under `crates/` after the post-restructure state (Phases 1–7 complete):

- `picoem-common` — shared primitives crate: `Memory`, `ClockTree` math, portable `Pacer`, PIO primitive types (`PioBlock`/`StateMachine`), divider/FIFO, and portable `threaded::{SpinBarrier, SpscQueue}` primitives.
- `picoem-devices` — off-chip device models used by harnesses and apps: `psram.rs`, `lcd.rs`, `i2s_capture.rs`. Depended on by `rp2040-emu` (PSRAM compat).
- `rp2350-emu` — the RP2350/RP2354 emulator library (dual Cortex-M33 + RISC-V Hazard3 cores, bus, memory, SIO, PIO, DMA, full peripheral suite, clocks, pacer, threaded runtime).
- `rp2350-emu-tui` — the TUI demo app driving `rp2350-emu`.
- `rp2040-emu` — the RP2040 emulator library (dual Cortex-M0+ cores, bus, memory, SIO, PIO, clocks, threaded runtime).
- `rp2040-emu-tui` — the TUI demo app driving `rp2040-emu`.
- `picoem-harness` — differential test binaries (QEMU diff + probe-rs diff variants per chip, softfloat diff, paced benchmark, silicon oracles, PicoGUS replay, OneROM oracles, test_silicon orchestrator).
- `picoem-debug` — GDB RSP scaffolding (stub).
- `epio-sys` — native FFI for the reference PIO simulator. **Excluded from workspace `default-members`** because it requires `clang` and vendored submodules; build explicitly with `cargo build -p epio-sys`.

See `wrk_docs/2026.04.14 - HLD - mdpicoem Workspace Restructure.md` for the phase-by-phase restructure plan.

## Build & Test

```bash
# Build everything
cargo build --release

# Run all unit tests
cargo test

# Run a single test (substring match)
cargo test <test_name_substr>

# Run tests in one crate
cargo test -p rp2350-emu

# Code coverage
cargo llvm-cov
```

## Differential Fuzz Testing (QEMU harness)

Two QEMU differential oracles, one per chip:

- **`qemu_diff_m33`** (the RP2350/RP2354 oracle) spawns a QEMU Cortex-M33 on GDB port `3333`, runs the same instruction in both QEMU and `rp2350-emu`, and diffs R0–R15 + xPSR (with masking for architecturally unpredictable flag fields).
- **`qemu_diff_m0plus`** (the RP2040 oracle) spawns a QEMU `cortex-m0` on GDB port `3334` (QEMU 10.2 has no `cortex-m0plus` model; M0+ is a strict ISA superset of M0 for the subset under test — see `tech_debt.md`), runs the same instruction in both QEMU and `rp2040-emu`, and diffs the same state.

```bash
# RP2350 / Cortex-M33 oracle
cargo run -p picoem-harness --release --bin qemu_diff_m33 -- --fuzz <N>
cargo run -p picoem-harness --release --bin qemu_diff_m33 -- --fuzz <N> --seed <S>
cargo run -p picoem-harness --release --bin qemu_diff_m33             # edge cases only

# RP2040 / Cortex-M0+ oracle
cargo run -p picoem-harness --release --bin qemu_diff_m0plus -- --fuzz <N>
cargo run -p picoem-harness --release --bin qemu_diff_m0plus -- --fuzz <N> --seed <S>
cargo run -p picoem-harness --release --bin qemu_diff_m0plus          # edge cases only
```

### Typical fuzz sessions

| Goal | Command |
|---|---|
| Quick smoke test | `--fuzz 1000` |
| Standard session | `--fuzz 100000` |
| Extended soak | `--fuzz 1000000` (or more) |

When asked to "fuzz test" or "do some fuzzing", default to `--fuzz 100000` unless a different count or duration is specified. For time-based requests ("fuzz for 2 hours"), estimate iterations from prior run throughput and adjust.

On `test_rp2350_qemu_diff_riscv32`, `--fuzz N --class X` now means "N cases of class X" (pre-dispatch); pre-2026-04-18 it meant "N mixed cases post-filtered to class X" (so low-weight classes produced far fewer than N). Seed reproducibility for `--fuzz N` without `--class` (the regression-gate path) is byte-identical across this change.

### Handling failures

When the harness reports a mismatch:
1. Note the seed and instruction class from the failure output.
2. Reproduce with `--seed <S>` to get a deterministic repro.
3. Investigate the specific instruction's decode/execute path in our emulator.
4. Fix and re-run the same seed to confirm.

### Running alongside concurrent builds (Windows)

Windows holds an exclusive lock on a running `.exe`. While `qemu_diff_m33.exe` / `qemu_diff_m0plus.exe` is fuzzing, any link step that tries to overwrite *that specific binary* — workspace-wide `cargo build --release`, or `-p picoem-harness` — will fail with an access-denied linker error. This blocks other agents rebuilding the harness.

Scope is narrow: builds and tests that don't touch the harness binary (e.g. `cargo build -p rp2350-emu`, `cargo test -p rp2040-emu`) are unaffected.

When starting a long fuzz run, copy the binary first so `target/release/<bin>.exe` stays free:

```bash
cp target/release/qemu_diff_m33.exe /tmp/fuzzer.exe
/tmp/fuzzer.exe --fuzz 100000
```

The overnight drivers under `fuzz-runs/` (`run-m33.sh`, `run-m0plus.sh`, `run-probe.sh`) already do this — they copy the harness to `fuzz-runs/<bin>.<pid>.exe` at startup and delete it on exit.

## Workspace Layout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0x4D44/picoem](https://github.com/0x4D44/picoem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
