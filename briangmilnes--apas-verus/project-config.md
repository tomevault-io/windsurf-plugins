---
trigger: always_on
description: This file is the single source of truth for AI assistant behavior on this project.
---

# CLAUDE.md — APAS-VERUS Project Rules

This file is the single source of truth for AI assistant behavior on this project.
It was synthesized from `.cursor/rules/` (80+ rule files).

---

## Project Overview

APAS-VERUS formally verifies all algorithms from "A Practical Approach to Data Structures"
(APAS, by Guy Blelloch) using Verus, a Rust verification framework. The primary objective
is to get code to **verify (prove)** with Verus.

- **Read ALL standards before writing or modifying any code.** At the start of every
  task, read every file in `src/standards/`. They total ~6200 lines (~54K tokens) — under
  6% of your 1M context. There is no excuse for skipping them. If a prompt says "pay close
  attention to standard N", that standard is especially critical for your task, but you must
  still read all of them. Agents that skip standards write code that violates project
  conventions and has to be reverted — this has happened repeatedly and wastes rounds.

### Standards Index

| # | Standard | Read when... |
|---|----------|-------------|
| 1 | `mod_standard.rs` | Creating or restructuring a module |
| 2 | `view_standard.rs` | Adding or modifying a `View` impl |
| 3 | `deep_view_standard.rs` | Adding or modifying a `DeepView` impl |
| 4 | `spec_wf_standard.rs` | Adding or modifying `spec_wf` predicates |
| 5 | `spec_naming_convention.rs` | Naming any spec function |
| 6 | `multi_struct_standard.rs` | Working with tree/enum types (multiple structs + enum) |
| 7 | `partial_eq_eq_clone_standard.rs` | Adding PartialEq, Eq, or Clone impls |
| 8 | `using_closures_standard.rs` | Writing code with closures, `Fn`, filter, map, tabulate, join |
| 9 | `total_order_standard.rs` | Writing ordering specs (min, max, find, rank, sorted) |
| 10 | `iterators_standard.rs` | Adding iterators to a collection |
| 11 | `wrapping_iterators_standard.rs` | Wrapping an existing iterator |
| 12 | `table_of_contents_standard.rs` | Reordering sections in a file |
| 13 | `mut_standard.rs` | Working with `&mut` parameters in Verus |
| 14 | `arc_usage_standard.rs` | Using `Arc` in verified code |
| 15 | `hfscheduler_standard.rs` | Using HFScheduler for fork-join parallelism |
| 16 | `toplevel_coarse_rwlocks_for_mt_modules.rs` | Writing Mt modules with RwLock |
| 17 | `tsm_standard.rs` | Thread-safe memory patterns |
| 18 | `finite_sets_standard.rs` | Working with finite sets in specs |
| 19 | `helper_function_placement_standard.rs` | Placing helpers in traits vs. free functions |
| 20 | `using_rand_standard.rs` | Using randomness in verified code |
| 21 | `using_hashmap_standard.rs` | Replacing std::collections::HashMap with verified types |
| 22 | `capacity_bounds_standard.rs` | Integer max bounds in requires for insert/push/resize |
| 23 | `mt_type_bounds_standard.rs` | Mt trait aliases (StTInMtT, MtKey, MtReduceFn, MtPred, MtMapFn, etc.) |
| 24 | `no_unsafe_standard.rs` | No unsafe — no `unsafe impl`, `unsafe fn`, or `unsafe {}` blocks |
- Run `scripts/validate.sh` after making changes
- Fix verification errors before moving on
- **NEVER run linters, formatters, or auto-fix tools.** No `cargo fix`, no `rustfmt`,
  no `cargo clippy --fix`, no auto-formatting of any kind. These tools revert proof work
  and destroy hours of edits. Only run `scripts/validate.sh`, `scripts/rtt.sh`, and
  `scripts/ptt.sh`. If a pre-commit hook runs a linter, investigate and disable it —
  do NOT let it rewrite source files.
- Prefer verified code over unverified code, even if it requires restructuring
- **Never sequentialize parallel files**: Mt (multi-threaded) implementations must remain
  parallel. Do not replace threaded code with sequential loops to satisfy the verifier.
- **Never propose serializing Mt algorithms** without exhausting all options for verified
  parallelism AND getting explicit user approval. The default is **no**.
- **Nothing is permanently blocked.** We can prove ALL of APAS-VERUS. Do not label any
  chapter, file, or proof obligation as "permanently" unverifiable. If a proof is hard,
  say it is hard — not that it is impossible. Every `assume`, every `external_body` on
  algorithmic logic, every weak spec is a target, not a fixture.
- **Skip Example and Problem files unless explicitly assigned.** Files named
  `Example*.rs` or `Problem*.rs` are textbook demos or problem sets, not algorithmic
  implementations. Do not spend time proving holes in them unless the user or your
  prompt explicitly directs you to. Do not include them in hole counts or proof targets.
  The proof effort belongs on the real algorithm files.
- **Algorithm files CAN get RTTs.** Files named `Algorithm*.rs` (e.g.,
  `Algorithm21_1.rs`) contain real executable code and should get runtime tests.
  Example and Problem files should NOT get RTTs.

### Abbreviations

| Abbreviation | Meaning |
|---|---|
| PTT / ptt | Proof Time Tests (Verus verification tests in `rust_verify_test/`) |
| RTT / rtt | Run Time Tests (Rust cargo tests in `tests/`) |
| DOT | Don't Over Think |
| RMF | Read My File (re-read before acting) |
| WN | What's Next |
| TUS | The Usual Suspects (search locations) |
| PBOGH | Project's current state (not something to hide) |
| AFK | Away From Keyboard (execute without stopping) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briangmilnes/APAS-VERUS](https://github.com/briangmilnes/APAS-VERUS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
