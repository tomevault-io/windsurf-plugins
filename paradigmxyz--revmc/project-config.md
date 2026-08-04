---
trigger: always_on
description: `--all-features` does not work. Just use default features and the following commands:
---

# revmc - EVM JIT/AOT compiler

## Commands

`--all-features` does not work. Just use default features and the following commands:

```bash
cargo cl                                                      # lint
cargo fmt --all                                               # format
cargo docs                                                    # check docs

cargo nextest run --workspace                                 # test all
cargo nextest run --workspace "test_name"                     # test single
cargo nextest run --workspace "statetest"                     # test statetests
SUBDIR=stRevertTest cargo nextest run --workspace "statetest" # test single statetest
```

## Architecture

- `revmc` — thin umbrella crate that re-exports codegen and runtime APIs.
- `revmc-codegen` — EVM compiler, bytecode analysis, linker, and compiler test infrastructure.
- `revmc-runtime` — runtime JIT/AOT backend, worker pool, artifact store, and revm integration.
- `revmc-backend` — abstract compiler backend trait. `revmc-llvm` is the main implementation.
- `revmc-builtins` — runtime builtins called by JIT-compiled code (host calls, gas accounting).
- `revmc-context` — EVM execution context types bridging revm and compiled code.
- `revmc-build` — build-script helpers for AOT compilation.
- `revmc-cli` / `revmc-cli-tests` — CLI frontend and its integration tests.
- `revmc-statetest` — Ethereum state test runner.

## CLI

Do NOT use `--release` — dev profile already uses `opt-level = 3`, and release
strips debug info and uses LTO which makes builds much slower for no benefit
during development.

```bash
cargo r -- run --list                  # list available benchmarks
cargo r -- run usdc_proxy              # compile and run a benchmark
cargo r -- run usdc_proxy -o tmp/dump  # compile and run a benchmark; dump files like opt.ll, remarks.txt to tmp/dump
cargo r -- run usdc_proxy --parse-only # parse and analyze only (no codegen)
cargo r -- run usdc_proxy --display    # print parsed bytecode IR
cargo r -- run usdc_proxy --dot        # render CFG as DOT/SVG
cargo r -- run usdc_proxy --aot        # compile to shared library
cargo r -- run 0x6001600201            # run custom bytecode (hex)
cargo r -- run 'PUSH1 1 PUSH1 2 ADD'   # run custom bytecode (asm string)
```

`-o <dir>` writes dumps under `<dir>/<benchmark>/`. Common files:

- `bytecode.bin` — raw input bytecode.
- `bytecode.txt` — parsed bytecode IR with blocks, gas, stack info, and comments.
- `bytecode.dbg.txt` — verbose debug dump of the parsed bytecode structure.
- `bytecode.dot` / `bytecode.svg` — rendered CFG.
- `unopt.ll` — LLVM IR before optimization.
- `opt.ll` — optimized LLVM IR.
- `opt.s` — final optimized assembly.
- `remarks.txt` — compile timings, JIT size, and generated-file sizes.

Use `RUST_LOG` to control log output:

```bash
RUST_LOG=debug cargo r -- run usdc_proxy   # all debug logs
RUST_LOG=revmc=debug cargo r -- ...        # only revmc crate logs
RUST_LOG=revmc::bytecode=trace cargo r --  # trace a specific module
```

## Injecting LLVM args

Extra LLVM command-line arguments can be passed via the `REVMC_LLVM_ARGS`
environment variable (space-separated):

```bash
REVMC_LLVM_ARGS="-debug-only=isel" cargo r -- run usdc_proxy
REVMC_LLVM_ARGS="-print-after-all" cargo r -- run usdc_proxy
```

LLVM args are a one-shot global (`LLVMParseCommandLineOptions`); only the first
call takes effect.

## Checking dynamic jump resolution

To get jump resolution stats across benchmarks:

```bash
./scripts/bench.py /tmp/bench --jump-resolution                    # all benchmarks
./scripts/bench.py /tmp/bench --jump-resolution usdc_proxy weth    # specific benchmarks
```

To inspect a single contract in detail:

```bash
RUST_LOG=debug cargo r -- run usdc_proxy --display |& rg 'jump|JUMP'
```

- `resolved jumps newly_resolved=N` — jumps resolved by block analysis.
- `unresolved dynamic jumps remain n=N` — jumps that couldn't be resolved.
- `JUMP bb<N>` / `JUMP bb<N>, bb<M>` — resolved (single/multi-target).
- `JUMP               ; pc=<N>` (no `bb` target) — unresolved dynamic jump.

Use `cargo r -- run --list` to see available benchmark names.

## Benchmarking against another revision

`./scripts/bench.py` is the unified benchmarking tool. It collects codegen line
counts, compile times, jump resolution stats, and constant-input statistics.

The script writes its full markdown output to `<dump_dir>/results.md` in
addition to printing it to stdout. Summary tables hide changes within a noise
threshold (1% for codegen, 5% for compile times); the `<details>` tables still
show every change.

```bash
./scripts/bench.py /tmp/bench --diff main                          # codegen + compile time vs main
./scripts/bench.py /tmp/bench --diff main usdc_proxy seaport       # specific benchmarks
./scripts/bench.py /tmp/bench --diff main --extra-dir tmp/mainnet  # include mainnet .bin files
./scripts/bench.py /tmp/bench                                      # current branch only (no diff)
./scripts/bench.py /tmp/bench --diff main --compile-times          # compile times only
./scripts/bench.py /tmp/bench --diff main --codegen-lines          # codegen lines only
./scripts/bench.py /tmp/bench --jump-resolution                    # jump resolution stats

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paradigmxyz/revmc](https://github.com/paradigmxyz/revmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
