---
trigger: always_on
description: SVSM is an in-guest paravisor written in Rust and designed for secure
---

# Introduction

SVSM is an in-guest paravisor written in Rust and designed for secure
virtualization environments, primarily AMD SEV-SNP and Intel TDX. It operates
at the highest privilege level within the Confidential Virtual Machine (VMPL0
on SEV-SNP).

# Directory structure

The SVSM runs on bare metal as a ring-0 kernel, with optional userspace
components. Important directories:

* `kernel/`: main SVSM kernel
* `boot/`: kernel bootloader and definitions
* `syscall/`: SVSM kernel/userspace system call definitions
* `user/`: SVSM userspace library and example init process
* `tools/`: general tools

# Building and testing

The SVSM project is organized as a cargo workspace. Some workspace members
are built with the host toolchain and others are built for bare metal. Running
`cargo build` or `cargo test` will not necessarily work.

* Build: use `make`.
* Test: use `make test` for unit tests. Read
  `Documentation/docs/developer/TESTING.md`.
* Linter: run `make clippy`.
* Format checks: `cargo fmt --check` works as usual.

All of the above must run cleanly after making a set of changes. Additionally,
all `unsafe` blocks must have a `SAFETY` comment detailing why the code is safe
and/or what invariants are assumed.

Additional code documentation guidelines can be found at
`Documentation/docs/developer/RUSTDOC-GUIDELINES.md`.

## Build recipes

The project uses custom build tools to generate an image that may be loaded
by the hypervisor. The top-level orchestrator is the `xbuild` tool, which is
called from the Makefile. The recipes are located in `configs/`. Information
about the build recipe format can be found in
`Documentation/docs/installation/BUILD_RECIPES.md`.

## IGVM

IGVM is the image format that the build system generates and that the host
VMM accepts. It describes the memory contents of the guest at startup. The
SVSM generates the IGVM image via `tools/igvmbuilder`, which is orchestrated
by `xbuild`.

## Formal verification with Verus

This repository integrates formal verification using **Verus**.
* Verified source files end with `*.verus.rs` or `*.proof.verus.rs` (e.g.,
  `address.verus.rs`, `alloc_perms.verus.rs`).
* Verification Invariants: Never bypass, alter, or strip
  `#![cfg_attr(verus_keep_ghost, ...)]` blocks or Verus ghost specifications
  unless specifically instructed. Any structural change to a verified module
  requires re-verification by executing the Verus toolchain.
* If a compiler flag or configuration change is made, ensure the `verus` or
  `verus_all` feature configurations in `Cargo.toml` remain valid.
* Do not run any verification checks unless specifically asked OR if related
  code was updated.
* Usage documentation is found in `Documentation/docs/developer/VERIFICATION.md`.

## General coding guidelines

* Wrap accesses to guest memory with `GuestPtr`. This type makes sure that an
  access fault is handled and returned as a regular error.
* Be exceedingly careful when accessing guest and host-shared memory. In
  general, shared memory may contain any bit combination and can be updated at
  any moment. We use `FromBytes` to make sure a type has no invalid bitwise
  representations, and `Sync` to make sure that unsynchronized concurrent
  accesses do not break assumptions. Creating Rust references (`&T` / `&mut T`)
  to shared memory is never safe and must be avoided.

# Platform

This section describes the confidential computing platforms the SVSM runs on,
and their security model. The SVSM abstracts platform details under the
`SvsmPlatform` trait.

The host hypervisor (HV) is **untrusted**: it sits outside the Trusted
Computing Base (TCB) and can manipulate VMCB fields, intercept VM exits, and
control physical memory assignment, but it cannot read or tamper with encrypted
guest memory or forge attestation measurements.

Confidentiality and integrity are part of the security model, while availability
is not. The SVSM aims to be robust and stable, but panicking on invalid states
or unrecoverable conditions is allowed and expected.

## SVSM-guest communication

The SVSM provides services for the lower privilege guest via a shared memory
protocol. Guests request services by writing parameters to a shared memory
calling area and triggering a `VMGEXIT` (SEV-SNP) or a `TDVMCALL` (TDX) that
traps into SVSM at VMPL0 (or L1).

Each protocol has an ID, and each command available within that protocol has a
dedicated call ID. Supported protocols:

| ID | Name                  | Implementation                     | Purpose |
|----|-----------------------|------------------------------------|---------|
| 0  | Core Protocol         | `kernel/src/protocols/core.rs`     | Protocol discovery, memory validation requests (`PVALIDATE`), guest vCPU creation/deletion |
| 1  | Attestation Protocol  | `kernel/src/protocols/attest.rs`   | Provides attestation reports, signing certificates, and manifest querying |
| 2  | vTPM Protocol         | `kernel/src/protocols/vtpm.rs`     | Bridges simulated TPM commands to the TPM reference simulator |
| 3  | Virtual APIC Protocol | `kernel/src/protocols/apic.rs`     | Provides virtual local APIC registers, configuration, and vector settings |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coconut-svsm/svsm](https://github.com/coconut-svsm/svsm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
