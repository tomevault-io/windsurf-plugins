---
trigger: always_on
description: Navigational map + the non-negotiable rules. Read `README.md` and `ARCHITECTURE.md`
---

# CLAUDE.md — kayfabe (Mode-2 Rust rewrite)

Navigational map + the non-negotiable rules. Read `README.md` and `ARCHITECTURE.md`
first; the settled design lives in `../nvidia-gpu-passthrough/docs/design/`
(`mode2_rust_rewrite_architecture.md`, `mode2_rust_testing_strategy.md`,
`mode2_abi_agnostic_layer.md`) and the decisions memo `mode2_rewrite_design_decisions`.
The design is settled — **implement it, do not re-improvise architecture.**

## The two rules that define this repo

1. **The core is pure.** `kayfabe-core`, `-mmu`, `-fwd`, `-completion`, `-arch`, `-util`,
   `-completion` contain **no** OS calls, no syscalls, no real time, no hypervisor
   types, no `#[repr(C)]` NVIDIA wire structs, and **no concrete GPU-generation or
   driver-version name** (`Ampere`, `V580`, …). `#![forbid(unsafe_code)]` workspace-wide.
   Everything effectful crosses a trait: `Vmm`, `Arch`, `RmBackend`, `Isolate`.
   - **Quarantine (Axis A): a `#[repr(C)]` layout is either QUARANTINED or provably
     OUR OWN WIRE FORMAT — and foreign is the default.** The hazard is a layout whose
     *owner decides when it changes*, frozen into a crate with no version dispatch, so
     the question the CI gate asks is **"foreign, or own-wire?"**, never "is this crate
     on a list?".
     - NVIDIA's layouts → `kayfabe-abi`. The kernel's uapi → `kayfabe-linux-raw`.
     - **Own-wire** is admitted anywhere, and must be *proved* by two facts the gate
       checks: a structure of the **same name** in a repository-local `.h`, **and** a
       `tests/wire_mirror.rs` in that crate enforcing the pair in **both** directions.
       Both, or it is foreign. (`kayfabe-qemu-raw` is the first: its counterpart is
       `qemu/hw/misc/nvkvm/kayfabe_shim.h`, and the pair additionally carries an ABI
       number and a `sizeof` checked at runtime in both directions — stronger than the
       quarantine gives the NVIDIA structs.)
     - ★ This text used to read *"live ONLY in `kayfabe-abi`"* with the gate's failure
       message adding *"do not add a third exempt crate"*. That phrasing was replaced
       rather than exempted-around, on the owner's ruling: **lengthening an exemption
       list weakens a rule with zero red tests** and invites a fourth entry
       (`gates_quantified_over_a_list`, three prior instances). The predicate fails
       closed, so a new crate needs no CI edit and cannot arrive without both proofs.
   - Grep gate (CI): no `Ampere|Turing|Hopper|Blackwell|Ada|V5\d\d` in any logic crate.

2. **Arch impls inherit the core without editing it.** Adding a real GPU generation is
   `impl Arch for <Gen>` (+ maybe one `GmmuFmt`) in an adapter crate, with **zero edits
   to any logic crate**. If a change to support an arch/version/hypervisor requires
   touching `kayfabe-core`/`-mmu`/`-fwd`/`-completion`, the seam is wrong — fix the seam,
   not the core. `kayfabe-mocks::MockArch` is the standing proof (the whole suite runs the
   real core against a fake arch).

## Green-gate discipline (inherited from uwgsocks; testing strategy §7)

- **Iterate until green, then review.** `cargo build` + `cargo test` + `cargo clippy
  --all-targets` must be clean before a commit that claims a milestone.
- ★★★ **CI is opportunistic convenience, not the definition of green** (owner ruling,
  2026-07-30). The authoritative run is **`scripts/run_full_suite.sh`** on a real box: the
  whole `stable` job + the five other CI jobs + everything CI structurally cannot do (real
  KVM, real namespaces, the vendored ogkm trees, a real GPU), ending in a ledger where every
  skip is named with its unmet requirement. Exit 0 = *everything this box can run, ran*. The
  census — what each gated family requires, what it does when the requirement is absent, and
  what had never run **anywhere** — is `docs/reference/full_suite_on_real_hardware.md`.
  ⊘ Never turn a skip into a quiet pass to make a run clean; a named loud skip is the floor.
- **No merge on red.** A red unit/integration test blocks the commit.
- **Tests must be mean and hard.** The #14 mock MUST reproduce the identical-VA +
  identical-handle collision, not a sanitized version. A mock that resolves the loser's
  VA too easily is a bug in the test — flag and harden it.
- **Every C quirk becomes a named regression test** (`t12_*`/`t13_*`/`t14_*`,
  `taddr_*`, `cb*`) as its subsystem ports. A red there = regressing a fix that cost
  days. The full classification (impossible / tested / gap-deferred, per C bug) is
  `docs/design/c_bug_regression_matrix.md` (decision #18B) — extend it when a new
  subsystem ports or a new C incident lands.
- **Security is the highest bar** (priority ladder, decision #8): a boundary-1/2/3
  failure or a fuzz crash outranks every other signal.
- **How to write a test that means something** — non-vacuity, exact-variant assertions
  (never `is_err()`), composed runs vs isolated cases, and the gates that can be wrong
  *upward*: `docs/design/testing_doctrine.md`. Every rule there is a generalisation of a
  specific incident, cited.
- **An optimisation with a correct-but-slow fallback ships with that fallback as a
  FIRST-CLASS tested mode**, and the *transitions* between them are tested by randomised

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reindertpelsma/kayfabe](https://github.com/reindertpelsma/kayfabe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
