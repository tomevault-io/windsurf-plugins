---
trigger: always_on
description: > This file is written for AI coding agents. It assumes you know Rust and Cargo but nothing about this specific project.
---

# NetRaze — Agent Guide

> This file is written for AI coding agents. It assumes you know Rust and Cargo but nothing about this specific project.

---

## Project Overview

**NetRaze** is an offensive network-execution toolkit written in pure Rust. It is the spiritual successor to the Python tools NetExec / CrackMapExec — same workflow (enumerate, authenticate, execute, post-exploit), but rebuilt for:

- **Single static binaries** — no Python runtime or native extension hell.
- **Async I/O from the ground up** — `tokio` across the entire stack.
- **Memory-safe wire protocols** — SMB2, NTLMSSP, and DCE/RPC are re-implemented in Rust and validated byte-for-byte against Impacket-generated fixtures. No FFI to Impacket or Samba libraries.
- **Cross-platform attacker OS** — the long-term goal is that Linux and Windows are equally capable attacker platforms. Today, some post-exploitation modules are still Windows-native only.

**Status:** Alpha. SMB2 + NTLMv2 are the most mature protocols. The project is currently blocked on `FSCTL_PIPE_TRANSCEIVE` over SMB2 (Phase 3 of the cross-platform portage), which will unlock rewriting all Windows-native post-exploitation modules as pure-Rust code.

**License:** BSD-2-Clause.

---

## Technology Stack

- **Language:** Rust, edition 2024, MSRV 1.85.
- **Toolchain:** Stable Rust with `clippy` and `rustfmt` components (see `rust-toolchain.toml`).
- **Async runtime:** `tokio` (`rt-multi-thread`, `macros`, `signal`, `sync`, `time`).
- **CLI framework:** `clap` with derive macros.
- **Desktop GUI:** `egui` + `eframe` with the `wgpu` backend (not the default `glow` / OpenGL). `wgpu` is configured with `dx12`, `vulkan`, `metal`, `gles`, and `wgsl` features for cross-platform and headless/WSL compatibility.
- **Serialization:** `serde` + `serde_json`.
- **Diagnostics:** `tracing` + `tracing-subscriber`.
- **Crypto:** `aes`, `cbc`, `cipher`, `des`, `hmac`, `md-5`, `md4`, `rand`.
- **Graph / workflow UI:** `egui-snarl` (node graph), `egui_graphs`, `petgraph`.

---

## Workspace Structure

This is a Cargo workspace with 14 members (13 application crates + `xtask`).

### Crate Map

| Crate | Purpose | Key Notes |
|---|---|---|
| `netraze-core` | Domain contracts, shared types, traits, error types. | **Zero applicative dependencies.** Defines `ProtocolFactory`, `ModuleFactory`, `ScanRequest`, `Capability`, `NetRazeError`. |
| `netraze-app` | Composition root / service wiring. | The only crate allowed to know almost everything. Bootstraps registries, storage, output, config, and runtime. |
| `netraze-cli` | Thin CLI binary (`clap`). | **Must contain zero protocol logic.** Entry point for headless use. |
| `netraze-desktop` | `egui`/`eframe` GUI with node-graph workflow canvas. | Binary crate. Uses `wgpu` backend and `egui-snarl` for visual workflows. |
| `netraze-protocols` | Wire-level protocol handlers. | SMB is the only significantly implemented protocol. Others (LDAP, SSH, WinRM, RDP, FTP, MSSQL, NFS, VNC, WMI) are scaffold-only. |
| `netraze-dcerpc` | Pure-Rust DCE/RPC v5 stack. | NDR20, PDU framing, NTLMSSP auth verifier, MS-SRVS interface. No `cfg(windows)` allowed inside this crate. |
| `netraze-modules` | Post-exploitation module registry. | Categories: `active_directory`, `credentials`, `reconnaissance`. |
| `netraze-auth` | Credential types and authentication methods. | `CredentialSet`, `SecretMaterial`, `AuthMethod`. |
| `netraze-targets` | Target parsing and normalization. | Detects hostnames, IPs, CIDRs, file lists, Nmap XML, Nessus files. |
| `netraze-config` | App / workspace / runtime configuration. | `AppConfig`, `WorkspaceConfig`, `RuntimeConfig`, `LoggingConfig`. |
| `netraze-storage` | Workspace persistence trait. | Async trait `WorkspaceStore`. In-memory impl today; SQLite backend planned. |
| `netraze-output` | Console reporting and output events. | `OutputEvent`, `Reporter` trait, `ConsoleReporter` bridges to `tracing`. |
| `netraze-runtime` | Concurrency, timeouts, async orchestration. | `RuntimeProfile` with bounded thread limits. |
| `xtask` | Build automation stub. | Currently a placeholder. Cargo alias `cargo xtask` maps to `cargo run -p xtask --`. |

### Dependency Rules (enforced in code review)

- `netraze-core` depends on **no** applicative crate.
- `netraze-cli` contains **no** protocol logic.
- `netraze-protocols` and `netraze-modules` depend only on `netraze-core` (and transversals), never on the CLI.
- `netraze-app` is the **only** crate allowed to know almost everything.
- Shared logic ratchets *up* into `netraze-core` or a transversal crate — never stays buried in a protocol crate.

### Dependency Graph (simplified)

```
netraze-cli        netraze-desktop
      │                  │
      └────┬─────────────┘
           │
      netraze-app
           │
    ┌──────┼──────┬────────┬────────┬─────────┬──────────┐
    │      │      │        │        │         │          │
netraze-  netraze-  netraze-  netraze-  netraze-  netraze-  netraze-  netraze-
protocols modules   dcerpc    auth      targets   config    output    runtime
   │                                              │
   │                                         netraze-storage
   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xr1l3s/NetRaze](https://github.com/0xr1l3s/NetRaze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
