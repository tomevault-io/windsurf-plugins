---
trigger: always_on
description: k23 is a research Rust microkernel that runs WebAssembly as its primary execution environment. RISC-V 64 is the current target; aarch64 and x86_64 are in progress. Built with BUCK2 (not Cargo workspaces). `no_std` throughout; custom allocator (talc), async runtime (`kasync`), panic/unwinding runtime.
---

# AGENTS.md

k23 is a research Rust microkernel that runs WebAssembly as its primary execution environment. RISC-V 64 is the current target; aarch64 and x86_64 are in progress. Built with BUCK2 (not Cargo workspaces). `no_std` throughout; custom allocator (talc), async runtime (`kasync`), panic/unwinding runtime.

## Orientation

New here? Read this section, then `manual/src/overview.md` for the full architecture.

**Mental model.** Boot flows loader → kernel → WebAssembly guests. `sys/loader/` runs first (it's a UEFI application): it verifies the kernel image, maps it, sets up MMU state, and hands off to `sys/kernel/`, which brings up trap handling, memory, the `kasync` runtime (`sys/async/`), and ultimately runs WASM. Reusable, kernel-agnostic pieces live in `lib/` as standalone crates; `sys/` holds the parts that only make sense as *this* kernel. Before adding anything, ask "could this be a `lib/` crate?" first.

**Where to start reading.**

- `sys/loader/src/main.rs` — the first code that runs; boot, image verification, MMU setup, handoff.
- `sys/kernel/src/main.rs` — kernel entry; wires up `arch/`, `mem/`, `wasm/`, tracing, and the shell.
- `sys/async/src/lib.rs` — the `kasync` executor, `block_on`, and sync/time primitives.
- `lib/` — look here before hand-rolling a data structure; `wavltree`, `range-tree`, `arrayvec`, `spin`, `sharded-slab`, `cpu-local`, `mem-core` already exist.
- `manual/src/` — authoritative docs: `overview.md`, `startup.md`, `arch/` (memory layout, KASLR), `contributing/`.

**How work gets done.**

- **nix** provides the toolchain — always work inside `nix develop .#default`.
- **`just`** is the entry point to everything (build, test, lint, run). Skim the `justfile`; `just preflight` is the one gate that mirrors CI.
- **BUCK2** is the build system, not Cargo. Targets look like `//sys:k23-riscv64-qemu`. After adding/removing a crate, run `just rust-project` so rust-analyzer picks up the new graph.
- **QEMU** runs the kernel — `just run //sys:k23-riscv64-qemu` boots it interactively; `just selftests` runs the `.wast` suite under it.
- Multi-arch: the default lane is the host; prefix recipes with `platform=//platforms:riscv64` to target RISC-V. See `platforms/README.md`.

**Conventions that bite** (details in the sections below): `no_std` everywhere; a mandatory per-file license header; hand-written error types (no `thiserror`); `#[expect(…, reason = "…")]` over `#[allow]`; every `unsafe` block needs a `// Safety:` line; prefer property/fuzz/loom tests over example-based ones. The eight **Critical invariants** are load-bearing — skim them before touching traps, page tables, MMIO, atomics, the allocator, or the async core.

## Setup

Develop inside the nix devshell: `nix develop .#default`.

Toolchain is pinned in `rust-toolchain.toml` (nightly-2025-07-11). Don't bump it casually — nightly features (`allocator_api`, `asm_unwind`, `thread_local`, …) are load-bearing.

## Build, test, lint

`just preflight` is the single gate that mirrors CI.

```sh
just preflight                              # full local CI, host lane
just platform=//platforms:riscv64 preflight # same gate, riscv64 lane
just check                                  # quick compile-check
just clippy / check-fmt / fmt               # lint, format
just unittests / miri / loom [targets]      # host tests (miri = UB, loom = concurrency)
just selftests                              # boot kernel under QEMU, run .wast tests
just run //sys:k23-riscv64-qemu             # interactive kernel boot
```

`preflight` = `clippy` → `check-fmt` → `typos` → `check` → `unittests` → `miri` → `loom` → `selftests` → `buck2 audit` → `cargo-deny` → `reindeer-clean` → `check-license-headers`. Don't skip steps; license-header and reindeer checks fail PRs that pass locally otherwise.

After adding/removing a crate, run `just rust-project` so rust-analyzer picks up the new BUCK graph.

**Prefer tests that explore the input space** — `proptest`, fuzz targets, loom — over example-based tests. See `manual/src/contributing/adding-tests.md`.

## Repo map

| Path | What |
|---|---|
| `sys/kernel/` | Ties subsystems into a complete kernel: trap handling and glue that can't live elsewhere. Anything extractable belongs in `lib/` or `sys/async/`. |
| `sys/loader/` | Verifies the kernel image, maps it, sets up MMU state, hands off. |
| `sys/async/` | `kasync` — cooperative async runtime (executor, Park/Notify, time, sync primitives). |
| `lib/` | First-party reusable crates (`riscv`, `trap`, `uart-16550`, `cpu-local`, `mem-core`, `wavltree`, `unwind`, `panic-unwind`, `spin`, `fdt`, …). New subsystems start here unless irreducibly kernel-internal. |
| `build/` | Toolchain wiring, kernel-config DSL (`kcfg.bzl`), QEMU/loom/fuzz/bench harnesses. |
| `platforms/` | Target constraints (`//platforms:riscv64`, `:aarch64`, `:x86_64`). |
| `third-party/` | Vendored deps; `third-party/BUCK` is generated by `reindeer buckify` — don't hand-edit. |
| `tests/` | Shared `.wast` fixtures. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JonasKruckenberg/k23](https://github.com/JonasKruckenberg/k23) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
