---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Tilck is an educational monolithic kernel designed to be Linux-compatible at
binary level. It runs on i386 (primary), riscv64, and x86_64. It implements
~100 Linux syscalls and runs mainstream Linux programs (BusyBox, Vim, TinyCC,
Micropython, Lua) without custom rewrites. ~13,300 lines of kernel code.
Licensed BSD 2-Clause.

## Originality and licensing (BSD-2-Clause)

Tilck must remain 100% original work. **Never copy implementation code,
identifiers, struct layouts, or macro idioms from Linux, glibc, or any
GPL/copyleft/incompatibly-licensed source.** High-level *ideas* and
well-known techniques (exception tables, scheduler designs, etc.) are fine to
learn from and reimplement — the concrete code must be written from scratch in
Tilck's style. When borrowing a technique from Linux/FreeBSD, give it a
Tilck-native name and a from-scratch implementation; don't mirror their
identifiers (`__ex_table`, `fixup_exception`, `pcb_onfault`, ...). If unsure
whether something crosses from idea into copying, stop and ask.

## Boot time and runtime latency are non-negotiable

Target: embedded systems with hard-realtime ambitions. Evaluate
proposals against that future, not "it's educational today".

**Boot time is sacred.** Anchor numbers under pure QEMU emulation
(TCG, no KVM): full boot through the custom bootloader **under 100
ms**; loaded with `-kernel` **under 50 ms**. Order-of-magnitude
budget:

- **100 ns to ~1 us**: free. Initialize a kcond, install an IRQ
  handler, zero a struct at boot even if the resource isn't used
  yet. Don't contort code with lazy-init / one-time CAS /
  constructor-on-first-use to save a `kcond_init`.
- **~10 us to ~1 ms**: prefer to defer, not a hard rule. If the
  work is needed during boot anyway, just do it.
- **>1 ms, especially seconds**: hard rejection. Do the work async
  (kthread, worker job, lazy on first real use) or drop the
  feature.

Mistake-to-not-repeat: proposing a synchronous busy-wait for the
RTC second edge in `init_system_time()` to get a precise
`boot_timestamp` — would cost up to 1s of boot on the critical
path, for a feature an async kthread already covers. Right shape:
make the kthread simpler/faster (RTC UIE instead of polling), don't
drag work into boot.

Same principle for runtime hot paths (timer IRQ, scheduler tick,
syscall entry/exit, context switch, IRQ handlers). Don't add work
to a hot path to make a cold path cleaner.

## Working with Git History

**Check `docs/annotated-commit-history.txt` before `git log`/`show`/`blame`.**
Plain-text, 80-col walk of every first-parent commit on master
(~4,900 commits, March 2016+); each entry has the verbatim log
header plus an "AI notes:" summary of the diff, large refactors
grouped into arcs. Read via `less`, `/`-search for subsystem or
hash. Falls through to raw git only when annotation is too shallow.

`scripts/dev/claude/annotate_commits` builds and extends it
(sub-commands: regen-list / meta-range / merges / merge-log /
append / show-head / sha-meta).

## Your Dedicated Tools Directory: `scripts/dev/claude/`

Dedicated home for helper scripts I write for my own work. When a
task needs many related shell ops (git plumbing, file scans, repeated
pipelines), write a sub-command dispatcher here instead of one-shot
commands — the user grants permission once for
`Bash(scripts/dev/claude/<tool>:*)` and the workflow stays auditable.
Canonical pattern: `scripts/dev/claude/annotate_commits`
(argument-parsed sub-commands, single permission pattern, dev-only).

## AI-generated plans and specs: `docs/plans/`

Plans, designs, and feature specs I generate go in `docs/plans/` (kept for
history). The root-level `docs/*.md` (contributing, building, scheduler, etc.)
are the human-facing docs — keep them very readable and uncluttered; never drop
an AI working document at `docs/` root.

## Build Commands

```bash
# First-time per arch (i386 | riscv64 | x86_64)
export ARCH=i386
./scripts/build_toolchain                # cross-compiler toolchain
./scripts/build_toolchain -s host_gtest  # unit test deps
./scripts/build_toolchain {-l,-h}        # list / help

# Configure (cmake_run forwards to cmake; runs from root or out-of-tree)
./scripts/cmake_run                      # default debug, ARCH=i386
./scripts/cmake_run -DRELEASE=1          # -O3
./scripts/cmake_run -DDEBUG_CHECKS=0
./scripts/cmake_run -DARCH={riscv64,x86_64}
./scripts/cmake_run --contrib            # clang + -Wconversion + stress opts
./scripts/cmake_run --gcov               # kernel + unit-test coverage
./scripts/cmake_run --intr --gcov        # flags are independent + combinable
./scripts/cmake_run -h                   # list cmake_run's own flags

# Build (cmake_run auto-runs if needed)
make             # one file at a time, good for debugging
make -j          # parallel (don't use -j$(nproc))
make gtests      # unit tests (needs gtest/gmock)
make rem         # rebuild image only (drops fatpart + tilck.img)

# Out-of-tree
mkdir build-dir && cd build-dir && /abs/path/scripts/cmake_run && make -j

# All configs (slow, useful before push)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vvaltchev/tilck](https://github.com/vvaltchev/tilck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
