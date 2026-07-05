---
trigger: always_on
description: Guidance for Claude when working in this repository. Keep this file accurate when the project structure, build steps, or invariants change.
---

# CLAUDE.md

Guidance for Claude when working in this repository. Keep this file accurate when the project structure, build steps, or invariants change.

## What this project is

**SonicDPI** is a cross-platform DPI-bypass engine for YouTube and Discord (including Discord voice / UDP RTP), written in Rust and dual-licensed MIT OR Apache-2.0. It is a userspace + kernel-hook hybrid: an OS-specific interceptor catches packets at the network layer and a platform-agnostic engine decides what to do with them (split, fake, reorder, drop).

The repo is a single Cargo workspace (`Cargo.toml` at the root) with four crates under `crates/`.

## Crate layout and responsibilities

```
crates/
├── sonicdpi-engine/     ← platform-agnostic core. NO OS-specific code.
├── sonicdpi-platform/   ← per-OS packet interception (WinDivert / NFQUEUE / pf rdr-to).
├── sonicdpi-cli/        ← `sonicdpi` binary: clap subcommands, service install, profile loader.
└── sonicdpi-tray/       ← `sonicdpi-tray` binary: end-user system-tray on/off toggle.
                            Spawns sonicdpi.exe as a child — does NOT load WinDivert in-process.
```

Layering rule: **`engine` knows nothing about packets-on-the-wire mechanics, `platform` knows nothing about strategies, `cli`/`tray` glue them.** If you find yourself adding a `cfg(target_os = ...)` to the engine, you're in the wrong crate.

### sonicdpi-engine — the parts to know
- `Engine` (lib.rs) — top-level entry the platform backend talks to. Owns `FlowTable`, `StrategyPipeline`, `TargetSet`, `DnsCache`, `Profile`.
- `flow.rs` — `FlowKey` / `Flow` / `FlowTable`. Per-connection state. Hot path.
- `strategy.rs` — `Strategy` trait + `StrategyPipeline`. The five built-in strategies are: `tls-multisplit`, `fake-multidisorder`, `hostfakesplit`, `quic-fake-initial`, `discord-voice-prime`.
- `tls.rs` / `dns.rs` — minimal protocol parsers (ClientHello SNI extraction, DNS A/AAAA cache for SNI→IP correlation).
- `packet.rs` / `builder.rs` — IPv4/IPv6 + TCP/UDP packet representation and rebuild via `etherparse` with correct checksum recomputation.
- `fakes.rs` / `embedded_fakes.rs` — fake TLS ClientHello / QUIC Initial / STUN / Discord-shape payloads, generated with per-run randomness so the binary isn't fingerprintable by static signature.
- `fooling.rs` — TCP TTL/MD5SIG/badseq/badsum/timestamp tricks for fake-packet decoys.
- `proxy.rs` — userspace transparent-proxy mode used by macOS and the `Probe` CLI subcommand.
- `profile.rs` — TOML profile schema + `build_pipeline()`. Three built-ins: `youtube-discord`, `youtube-discord-seqovl`, `youtube-discord-hostfakesplit`.

### sonicdpi-platform — backends behind the `Interceptor` trait
- `windows.rs` — WinDivert. Uses the **EV-signed** WinDivert 2.2.2 binaries from `vendor/windivert/x64/` (see `.cargo/config.toml`). Do not switch to the `vendored` feature of `windivert-sys` — that crate's `cl.exe`-built dll has a stack-overflow bug in its filter parser. Build script auto-copies `WinDivert.dll` next to `sonicdpi.exe`.
- `linux.rs` — Pure-Rust `nfq` crate (no GPL `libnetfilter_queue` linkage). Installs nftables rules at startup. Requires `cap_net_admin` (or root). `iptables-only` hosts (kernel < 6.17) are not supported.
- `macos.rs` — `pf rdr-to` redirects target ports to a transparent userspace proxy; `DIOCNATLOOK` ioctl recovers the original destination. **TCP-only in v0.2** — `IPPROTO_DIVERT` was removed from XNU. Discord voice on macOS is blocked on the NetworkExtension System Extension backend (tracked in `docs/macos-networkextension.md`).

### sonicdpi-cli — `sonicdpi` binary
Subcommands: `run`, `install`, `uninstall`, `profiles`, `show`, `probe`. `service.rs` handles winsvc / systemd unit / launchd plist generation.

### sonicdpi-tray — end-user toggle
`elevation.rs` checks for admin/root upfront, `engine_guard.rs` spawns and supervises the `sonicdpi` child, `icons.rs` embeds tray PNGs. Uses `tray-icon` + `winit`.

## Build, test, lint

```bash
# Workspace build (all crates)
cargo build --workspace
cargo build --workspace --release

# Tests — engine logic only (no privileged network ops in unit tests).
# Integration tests that need real interceptors are gated on `--ignored`.
cargo test --workspace --lib

# Required to pass CI:
cargo fmt --all -- --check
cargo clippy --workspace --all-targets    # treated as -D warnings via RUSTFLAGS
cargo deny check licenses                 # license allowlist in deny.toml
```

Toolchain: `rust-toolchain.toml` pins `stable` with `rustfmt` + `clippy` components. MSRV is **1.78** (set in workspace `Cargo.toml`).

### Platform-specific build deps
- **Windows**: nothing extra — WinDivert binaries are vendored at `vendor/windivert/x64/`. The `.cargo/config.toml` sets `WINDIVERT_PATH` and bumps the link-time stack to 8 MiB on `x86_64-pc-windows-msvc` and `i686-pc-windows-msvc`.
- **Linux** (CI + dev): `sudo apt-get install -y libnetfilter-queue-dev nftables`. nftables is a runtime dep too.
- **macOS**: nothing extra.

### CI matrix (`.github/workflows/ci.yml`)
- `fmt` — Linux only.
- `clippy` — Linux + Windows + macOS, `-D warnings`.
- `test` — same matrix, `cargo test --workspace --lib`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [by-sonic/sonicdpi](https://github.com/by-sonic/sonicdpi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
