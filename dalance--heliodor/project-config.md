---
trigger: always_on
description: RV64GC RISC-V processor core written in [Veryl](https://veryl-lang.org/). Boots
---

# Heliodor

RV64GC RISC-V processor core written in [Veryl](https://veryl-lang.org/). Boots
mainline Linux 5.15 (1/2/4-hart SMP) to SBI shutdown. See `README.md` for the
architecture overview, status, and roadmap; this file covers the development
workflow.

## Veryl Toolchain

Heliodor is written in Veryl and needs the Veryl compiler/simulator to build and
test. Veryl lives at `./veryl/` as a **local clone** of veryl-lang/veryl —
gitignored, *not* a git submodule, so heliodor stays self-contained (same policy
as the `riscv-tests` upstream under `test/riscv-arch-test/`). Two situations:

**(1) Standalone heliodor development** — heliodor is the top-level repo and
`./veryl/` is a CHILD clone you check out and build:

```bash
git clone https://github.com/veryl-lang/veryl.git veryl
cd veryl && cargo build --profile release-verylup   # fast release: no LTO, parallel codegen
```

Use that binary — `veryl/target/release-verylup/veryl` — for all development and
testing. Use the debug build (`cargo build`, `veryl/target/debug/veryl`) only
when debugging the compiler/simulator itself; `cargo build --release` is the slow
max-optimization build. You can patch the compiler in this clone and branch/PR it
upstream.

**(2) heliodor as a Veryl test fixture** — this tree is checked out INSIDE the
veryl repo (veryl is an ANCESTOR dir) to test the compiler against a real RV64GC
design. Do NOT clone veryl; the compiler under test is the surrounding repo's
build. The veryl test harness runs heliodor's scripts with `$VERYL` set to that
binary (heliodor's scripts honor `$VERYL`). The clone step in (1) does not apply.

Key commands (run the resolved `veryl` binary from the project root):

| Command       | Description                  |
|---------------|------------------------------|
| `veryl check` | Analyze (type check, lint)   |
| `veryl build` | Transpile to SystemVerilog   |
| `veryl fmt`   | Format source files          |
| `veryl test`  | Run native testbenches       |
| `veryl clean` | Remove build artifacts       |

Refer to `veryl/testcases/veryl/` for language feature examples.

## Directory Structure

```
heliodor/
├── CLAUDE.md
├── Veryl.toml              # Project configuration
├── src/
│   ├── core/               # Pipeline stages, decode, ALU, FPU, CSR, RS/ROB/RAT, SoC
│   ├── cache/              # I-Cache / D-Cache / shared L2 / memory_bus
│   ├── mmu/                # Sv39 MMU
│   ├── peripheral/         # UART, CLINT, PLIC
│   └── pkg/                # Shared packages & type definitions
├── tb/                     # Testbenches (Veryl native test)
├── test/                   # RISC-V ISA tests, hex programs, C benchmarks
├── doc/                    # Documentation
└── veryl/                  # Veryl compiler — local clone, gitignored (see Veryl Toolchain)
```

## Development Conventions

- **Language**: Veryl (transpiles to SystemVerilog)
- **Comments & documentation**: English
- **Testing**: Veryl native testbench. The simulator chooses a code-generation
  backend with `--backend` (default: `cc`). The old `--disable-jit` flag is gone.
  ```bash
  veryl test                          # default cc backend (emit C + compile), all tests
  veryl test --backend cranelift      # in-process Cranelift JIT
  veryl test --backend interpret      # IR tree-walking interpreter (slowest, reference)
  veryl test --backend-validate       # dual-run cc vs cranelift, panic on divergence
  veryl test --test test_dcache_lbu   # run a specific test only (faster for debugging)
  ```
  To catch simulator/codegen bugs early, cross-check a suspicious result on a
  second backend (e.g. `--backend cranelift` or `--backend interpret`): two
  independent backends agreeing is strong evidence the RTL, not the sim, is at fault.
- **Regression testing**: After modifying heliodor or veryl, run the multi-step regression:
  1. `veryl test` — fast tests **+ the arch suite** (~150 tests, seconds-to-~minute;
     the rv64ui/um/ua/mi/si arch tests in `tb/test_arch_common.veryl` and the
     rv64uf/ud FP tests in `tb/test_arch_fp.veryl` are NOT `#[ignore]` and run here,
     on the OoO core). Fix any failures before proceeding.
  2. `veryl test --ignored --test test_litmus_4hart` — P9.0 RVWMO litmus battery
     at N=4 (IRIW + 4-way barrier/bus stress; ~2.5M cycles). The N=2 battery
     (`test_litmus_2hart`) already runs in step 1. Any forbidden-outcome hit is
     a memory-model bug — see `test/litmus/litmus.S` for the tohost encoding.
  3. `veryl test --ignored --test test_soc_smp_linux_boot_2hart` — N=2 SMP Linux boot
     (~12.3M cycles, ~2 min). N=1 single-hart is `--ignored --test test_soc_linux_boot`
     (~9.1M cycles, ~40 s).
  4. `veryl test --ignored --test test_soc_smp_linux_boot_4hart` — N=4 SMP Linux boot
     (~16.6M cycles, ~10 min). Cycle counts drift with perf work — treat large
     unexplained jumps as regressions (N=4 has a ±1M timing band).
- **Microbenchmarks**: `tb/test_bench.veryl` has 5 `#[ignore]` perf tests
  (`test_dhrystone`, `test_bench_{memcpy,multiply,median}`, `test_coremark`)
  that print cycles + retired instructions (IPC = instret / cycles, frozen at
  `tohost`). NOT a regression gate — run
  individually, e.g. `veryl test --ignored --test test_dhrystone`. Current

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dalance/heliodor](https://github.com/dalance/heliodor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
