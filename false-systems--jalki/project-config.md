---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is jälki

jälki is an Ahti producer for kernel and runtime evidence. It has four pieces:

- **SDK** — `jalki-sdk-meta` (source of truth) plus language SDKs (`jalki-sdk-python`, future Go/Rust). Defines types and the wire protocol.
- **API** — daemon IPC over `/run/jalki/jalki.sock`, MCP server, and the producer contract with Ahti.
- **Logic** — the `Probe` trait, eBPF programs, BTF resolution, ring buffer management, self-filtering, and the embedded knowledge base. Turns raw kernel signals into validated evidence.
- **Ahti** — where evidence lives durably. Jälki MUST NOT keep a parallel datastore.

The three built-in TCP probes (`TcpConnect`, `TcpClose`, `TcpRetransmit`) are batteries-included defaults — the logic layer makes writing *any* fentry/fexit probe a matter of implementing one trait.

Current code implements logic + SDK + local API. The Ahti producer path is the v0 work in `docs/jalki/v0-scope.md`.

## Crate Structure

```
jalki/
├── jalki-common/     # no_std shared types — kernel + userspace
├── jalki-ebpf/       # eBPF programs — NOT a workspace member (separate build target)
├── jalki/            # userspace daemon + library
├── jalki-codegen/    # runtime BPF program generation from BTF (no C, no clang)
├── jalki-mcp/        # MCP server (JSON-RPC 2.0 over stdin/stdout)
├── jalki-sdk-meta/   # source of truth for SDK types, wire protocol, conformance tests
├── jalki-sdk-python/ # Python SDK (NOT a workspace member — pyproject.toml)
├── xtask/            # build orchestration (eBPF compilation)
├── knowledge/        # JSON knowledge base — compiled into binary via include_str!
├── specs/            # Luotain-compatible requirement specs (tested by oracle)
├── helm/jalki/       # Helm chart for k8s deployment
└── eval/oracle/      # standalone contract test suite — NOT in workspace
```

Workspace members: `jalki-common`, `jalki`, `jalki-codegen`, `jalki-mcp`, `jalki-sdk-meta`, `xtask`.

Non-workspace (built separately): `jalki-ebpf`, `jalki-sdk-python`, `eval/oracle`.

External dependency: `false-protocol` is a path dependency from `../ahti/false-protocol`.

## Build & Run

**Build order matters.** Always build eBPF first. Userspace will compile without it, but the daemon fails at runtime with a missing eBPF object.

```bash
# 1. eBPF first — always (requires nightly + Linux)
cargo run -p xtask -- build-ebpf [--release]

# 2. Userspace daemon (requires Linux — aya doesn't compile on macOS)
cargo build -p jalki

# 3. Regenerate SDK files if jalki-sdk-meta types/protocol changed
cargo run -p jalki-sdk-meta -- --lang python --out jalki-sdk-python/src/jalki/

# Run daemon (requires root or CAP_BPF + CAP_PERFMON)
sudo RUST_LOG=jalki=debug ./target/debug/jalki \
    --ebpf-path jalki-ebpf/target/bpfel-unknown-none/debug/jalki-ebpf \
    --sink stdout
```

### macOS Development

`cargo check --workspace` and `cargo test --workspace` **fail on macOS** because `aya` is Linux-only. The crates that depend on aya (`jalki`, `jalki-codegen`, `jalki-mcp`) cannot be compiled on macOS.

Crates that work on macOS:

```bash
cargo check -p jalki-common
cargo check -p jalki-sdk-meta
cargo check -p xtask
cargo test -p jalki-common                                     # event struct size tests
cargo test -p jalki-sdk-meta                                   # SDK meta tests
cargo test --manifest-path eval/oracle/Cargo.toml              # oracle contract tests (all 50 cases)
cargo test --manifest-path eval/oracle/Cargo.toml -- case_014  # single oracle case
cargo clippy -p jalki-common -p jalki-sdk-meta                 # lint what compiles
```

### Linux Development (full build)

```bash
cargo check --workspace
cargo test --workspace                          # all workspace tests
cargo test -p jalki-common                      # event struct size tests
cargo test -p jalki                             # userspace tests
cargo test --manifest-path eval/oracle/Cargo.toml  # oracle contract tests
```

## Key Crate Details

### jalki-common

- `no_std` — must stay no_std, shared with kernel space
- `#[repr(C)]` event structs: `TcpConnectEvent`, `TcpCloseEvent`, `TcpRetransmitEvent`
- Feature `userspace` enables `aya::Pod` impls
- Size tests lock down the BPF ABI — do not change struct sizes without updating tests

### jalki-ebpf

- Separate build target: `bpfel-unknown-none`, requires nightly Rust
- NOT in the workspace Cargo.toml — has its own
- Build with: `cargo run -p xtask -- build-ebpf [--release]`
- Three programs: `fexit/tcp_connect`, `fexit/tcp_close`, `fentry/tcp_retransmit_skb`
- Four BPF maps: three ring buffers (one per probe, 4MB each) + `PID_FILTER` HashMap

### jalki (userspace)

- Library + binary in one crate
- **Daemon mode** (no subcommand): loads eBPF, attaches probes, drains events, emits, serves IPC
- **CLI subcommands**:
  - `ask "question"` — KB search → auto-deploy → collect → interpret → answer
  - `watch <function>` — deploy probe, collect for N seconds, print events
  - `stream [function]` — live ndjson event stream
  - `list [--layer tcp]` — browse the knowledge base
  - `status` — show attached probes, event counts, drops

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [false-systems/jalki](https://github.com/false-systems/jalki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
