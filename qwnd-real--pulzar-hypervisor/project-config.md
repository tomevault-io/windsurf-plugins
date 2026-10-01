---
trigger: always_on
description: Operating manual for AI agents working in this repository. Read it fully before
---

# AGENTS.md — pulzar

Operating manual for AI agents working in this repository. Read it fully before
writing or changing any code. Every rule below is binding unless the user
explicitly overrides it in the current conversation.

Pulzar is a type-1 hypervisor booted via UEFI. It has two parts: a UEFI loader
application (`hv-loader`) and the hypervisor image proper, which the loader
maps and transfers control to. This file is not a project summary — it defines
how you must behave and what the code must look like.

## 1. Prime directives

1. **Never resolve ambiguity silently.** If the task, the requirements, or your
   own understanding admit more than one reasonable interpretation, STOP and
   ask the user before implementing. Present the interpretations you see and
   what each would mean. Guessing and "picking the sensible one" is a failure,
   even if the guess turns out right.
2. **Touch only what the task requires.** Do not modify, "improve", reformat,
   or refactor code outside the scope of the request. Every changed line must
   trace directly to the user's request. If you notice unrelated problems
   (dead code, a bug, an ugly API), report them — do not fix them unprompted.
3. **Reuse before you write.** Before implementing anything, check (a) whether
   this workspace already has a function/type/module that does it, and
   (b) whether a well-maintained crate already provides it. Never hand-roll an
   assembly wrapper, bit-manipulation helper, or data structure that an
   existing dependency or existing project code already offers, and never add
   a second function that duplicates an existing one. If you believe existing
   infrastructure is inadequate, say why and ask before replacing it.
4. **No unfinished work in committed code.** No `todo!()`, no `unimplemented!()`,
   no `// TODO` markers, no stubbed-out branches, no silently omitted cases.
   Work is either complete, or its remaining part is surfaced to the user as an
   explicit, named open question — never buried in the code.
5. **No shortcuts on quality gates.** You may not suppress, downgrade, or
   disable a warning or lint to make a build pass. The only exception is a lint
   that is genuinely wrong for legitimate low-level systems code and has no
   compliant alternative; see §5 for the required procedure.
6. **Act as a senior Rust systems engineer.** Simplicity first, DRY, idiomatic,
   production-ready, performance-conscious. If a senior engineer would call
   your solution overcomplicated, rewrite it before presenting it.

## 2. Workspace shape

```
pulzar/
├── AGENTS.md            ← this file (CLAUDE.md is a symlink to it)
├── Cargo.toml           ← workspace root; lints and shared deps live HERE
├── rust-toolchain.toml  ← pinned nightly; do not float the channel
├── rustfmt.toml         ← formatting policy (uses unstable options → nightly)
├── .cargo/config.toml   ← cargo aliases (`cargo xtask`)
└── crates/
    ├── drivers/          ← device drivers that interpose a guest's device MMIO
    │   └── nvme/         ← answers a guest's NVMe identify commands with spoofed identity
    ├── hv-loader/       ← UEFI application: first-stage loader for the hypervisor
    ├── hv-core/         ← UEFI application: hypervisor image (stub until the loader loads it)
    └── xtask/           ← host tool: stages boot media, provisions guest disks, runs QEMU
```

- `hv-loader` is a `no_std`/`no_main` UEFI PE application built for
  `x86_64-unknown-uefi`, using the rust-osdev `uefi` crate. It runs under
  firmware boot services; its job (eventually) is to locate, map, and jump into
  the hypervisor image.
- `hv-core` is today a UEFI PE stub standing in for the hypervisor image, so
  boot media and staging are exercised end to end. The real hypervisor core
  will move to a custom freestanding target with `-Z build-std` — this is why
  the toolchain is nightly.
- There is no workspace-wide default target: the two UEFI applications pin
  `x86_64-unknown-uefi` via `forced-target` (nightly `per-package-target`
  feature) and pull every library crate in as a UEFI dependency, while the
  libraries themselves are target-agnostic `no_std` crates that plain
  `cargo build` compiles for the host too — which is what lets
  `cargo test -p <crate>` run natively. `xtask` builds for the host as well.

## 3. Toolchain and build

Agreed decisions, encoded in the config files — change them only with explicit
user approval:

- **Nightly Rust, pinned to a date** in `rust-toolchain.toml` (never channel
  `"nightly"` floating). Required for future `-Z build-std` on the freestanding
  hypervisor target and for the unstable rustfmt options in use. To bump the
  pin: ask first.
- **Edition 2024, workspace-wide**, via `workspace.package`.
- **Dependencies are declared in `[workspace.dependencies]`** at the root and
  inherited by member crates with `{ workspace = true }`. `Cargo.lock` is
  committed.
- Member crates set `[lints] workspace = true`. Never define per-crate lint
  levels.

Canonical commands (rustup picks up the pinned toolchain automatically):

```sh
cargo build                      # produces target/x86_64-unknown-uefi/debug/hv-loader.efi
cargo clippy --all-targets       # must exit 0 with zero warnings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qwnd-real/pulzar-hypervisor](https://github.com/qwnd-real/pulzar-hypervisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
