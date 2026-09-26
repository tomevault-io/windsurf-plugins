---
trigger: always_on
description: Use the root commands for lint, format, and docs. Targeted JIT commands:
---

# evm2-jit - EVM JIT/AOT compiler

## Commands

Use the root commands for lint, format, and docs. Targeted JIT commands:

```bash
cargo nextest run -p evm2-jit-codegen --test codegen                             # compiler tests
cargo nextest run -p evm2-jit-codegen --test codegen "test_name"                 # single compiler test
EVM2_JIT_TEST_DUMP=1 cargo nextest run -p evm2-jit-codegen --test codegen "test_name" # dump single compiler test
cargo nextest run -p evm2-jit-runtime                                            # runtime tests
cargo st "statetests::devnet::jit"                                               # EEST JIT state tests
cargo st "blockchain_tests::devnet::jit"                                         # EEST JIT blockchain tests
```

## Architecture

- `evm2-jit` — thin umbrella crate that re-exports codegen and runtime APIs.
- `evm2-jit-codegen` — EVM compiler, bytecode analysis, linker, and compiler test infrastructure.
- `evm2-jit-runtime` — runtime JIT/AOT backend, worker pool, artifact store, and evm2 integration.
- `evm2-jit-backend` — abstract compiler backend trait. `evm2-jit-llvm` is the main implementation.
- `evm2-jit-builtins` — runtime builtins called by JIT-compiled code (host calls, gas accounting).
- `evm2-jit-context` — EVM execution context types bridging evm2 and compiled code.
- `evm2-jit-build` — build-script helpers for AOT compilation.

## CLI

Do NOT use `--release` — dev profile already uses `opt-level = 3`, and release
changes panic/debug behavior and makes local iteration slower. Use the
`profiling` profile when symbolized optimized builds are needed.

```bash
cargo cli list <fixture.json>             # list replay entrypoints
cargo cli replay --jit <fixture.json>     # replay through JIT
cargo cli replay --aot <fixture.json>     # replay through AOT
```

When a compiler dump directory is configured, common files are:

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
RUST_LOG=debug cargo cli replay --jit <fixture.json>
RUST_LOG=evm2_jit_codegen=trace cargo cli replay --jit <fixture.json>
```

## Injecting LLVM args

Extra LLVM command-line arguments can be passed via the `EVM2_JIT_LLVM_ARGS`
environment variable (space-separated):

```bash
EVM2_JIT_LLVM_ARGS="-debug-only=isel" cargo cli replay --jit <fixture.json>
EVM2_JIT_LLVM_ARGS="-print-after-all" cargo cli replay --jit <fixture.json>
```

LLVM args are a one-shot global (`LLVMParseCommandLineOptions`); only the first
call takes effect.

Use `EVM2_JIT_PASSES` to override the LLVM optimization pass pipeline for local
experiments.

## Checking dynamic jump resolution

To get jump resolution stats across benchmarks:

```bash
./scripts/jit/bench.py /tmp/bench --jump-resolution                    # all benchmarks
./scripts/jit/bench.py /tmp/bench --jump-resolution usdc_proxy weth    # specific benchmarks
```

To inspect a single contract in detail:

```bash
RUST_LOG=evm2_jit_codegen::bytecode=trace ./scripts/jit/bench.py /tmp/bench --jump-resolution usdc_proxy |& rg 'jump|JUMP'
```

- `local_jumps.*newly_resolved=N` - jumps resolved locally.
- `resolved non-adjacent jump` - jump target resolved outside adjacent layout.
- `resolved via PCR hint` - jump target resolved through a pushed-code-region hint.
- `unresolved dynamic jumps remain n=N` — jumps that couldn't be resolved.
- `JUMP bb<N>` / `JUMP bb<N>, bb<M>` — resolved (single/multi-target).
- `JUMP ; pc=<N>` — unresolved dynamic jump.

## Benchmarking against another revision

`./scripts/jit/bench.py` is the unified benchmarking tool. It collects codegen
line counts, compile times, jump resolution stats, and constant-input
statistics.

The script writes its full markdown output to `<dump_dir>/results.md` in
addition to printing it to stdout. Summary tables hide changes within a noise
threshold (1% for codegen, 5% for compile times); the `<details>` tables still
show every change.

```bash
./scripts/jit/bench.py /tmp/bench --diff <base-rev>                    # codegen + compile time vs base
./scripts/jit/bench.py /tmp/bench --diff <base-rev> usdc_proxy seaport # specific benchmarks
./scripts/jit/bench.py /tmp/bench --diff <base-rev> --extra-dir tmp/mainnet # include mainnet .bin files
./scripts/jit/bench.py /tmp/bench                                      # current branch only (no diff)
./scripts/jit/bench.py /tmp/bench --diff <base-rev> --compile-times    # compile times only
./scripts/jit/bench.py /tmp/bench --diff <base-rev> --codegen-lines    # codegen lines only
./scripts/jit/bench.py /tmp/bench --jump-resolution                    # jump resolution stats
./scripts/jit/bench.py /tmp/bench --input-stats                        # constant-input stats
./scripts/jit/bench.py /tmp/bench --block-stats                        # block stats

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alloy-rs/evm2](https://github.com/alloy-rs/evm2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
