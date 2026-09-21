---
trigger: always_on
description: This file provides shared guidance to AI coding agents working in this repository. It is the canonical content behind
---

# Repository AI Instructions

This file provides shared guidance to AI coding agents working in this repository. It is the canonical content behind
both `AGENTS.md` and `CLAUDE.md`, and `.ai/` is the canonical repo-owned directory for shared AI instructions and
skills.

New skills go in `.ai/skills/<name>/SKILL.md` - never directly under `.claude/skills/` or `.agents/skills/`. Those
tool-specific directories contain per-skill symlinks back to `.ai/skills/`. See the `ai-shared-layout` skill for the
full layout, the `*.local.*` convention for per-user state, and the symlink-restore procedure.

## Overview

This is the Braiins clock codebase - a Rust-based embedded system for a smart clock device with a web frontend. The
project consists of a modular Rust backend running on OpenWRT (ARMv7), a React/TypeScript frontend, and uses Wayland for
the display UI.

**📖 For detailed architecture information, see [`docs/architecture/overview.md`](docs/architecture/overview.md)** - This
contains comprehensive documentation of the display system, state management, gesture handling, and performance
characteristics.

## Architecture

### Backend Structure (Rust)

The backend is organized as a Cargo workspace with the following main components:

- **`bmc`**: Core BMC library that orchestrates all other components (audio, LED, display, button management, scheduler,
  upgrades). Contains the main business logic, configuration, and system management.
- **`bmc-openwrt`**: Main binary for the OpenWRT control board - integrates all hardware drivers and BMC core
  functionality for the actual device.
- **`bmc-mock`**: Mock binary for development/testing on x86_64 with simulated hardware.
- **`bmc-grpc`**: gRPC service definitions (protobuf in `bmc-grpc/proto/web/`) for frontend-backend communication.
- **`bmc-scheduler`**: Alarm scheduling and cron-like functionality.
- **`bmc-audio`**, **`bmc-led`**, **`bmc-button`**, **`bmc-gpio`**, **`bmc-kobject`**: Hardware abstraction layers.
- **`bmc-platform`**: Platform-specific abstractions.
- **`bmc-upgrade`**: Firmware upgrade management.
- **`bmc-shared/`**: Shared libraries (`stopwatch`, `time`, `utils`).
- **`bmc-net/`**: Networking crates, shared with bos-main:
  - **`bmc-net`**: the `NetworkManager` facade — network config, provisioning state machine, setup AP and captive portal
    — with the `openwrt` (UCI), `buildroot` and `mock` backends.
  - **`bmc-net-types`**: dependency-light value types (`MacAddr`, network protocol config, WiFi status/scan).
  - **`bmc-net-drv`**: interface enumeration plus the `WifiDriver` backends (`nl80211`, `esp32`).
  - **`bmc-net-dns`**: the `IiResolver` DNS/NTP resolver.
  - **`bmc-net-mdns`**: mDNS/DNS-SD advertisement of the device web UI and API.
  - **`bmc-net-observe`**: synchronous, read-only connectivity probes for OS-driven overlays.
  - **`bmc-net-diag`**: network diagnostics for the support archive (ifconfig, public IP, ping).
- **`bmc-support`**: Platform-agnostic support-archive engine, shared with bos-main — `SupportConfig`, streamed
  `SupportArchive`, the `SupportFilter` and `SupportExtension` traits and the archive formats.
- **`bmc-support-openwrt`**: The OpenWrt board's shared support-archive pieces — credential filters for its config
  layout, the Nix profile and `logread` extensions. Each binary assembles them into its own `SupportConfig`.

### Frontend Structure (TypeScript/React)

See `frontend/CLAUDE.md`

### Communication Layer

Frontend and backend communicate via gRPC-Web using Protocol Buffers defined in `bmc-grpc/proto/web/`. The backend runs
a tonic gRPC server with tonic-web middleware. Frontend uses ConnectRPC (Connect-Web) for type-safe RPC calls.

## Build System

This project uses **Nix Flakes** as the primary build system.

### Building Components

```bash
# Build frontend (prints output)
nix build -L .#frontend --print-out-paths --no-link

# Build OpenWRT binary (ARMv7 release)
nix build .#bmc-openwrt-armv7-glibc-release

# Build OpenWRT binary (ARMv7 debug)
nix build .#bmc-openwrt-armv7-glibc-debug
```

### Development with Nix

```bash
# Enter default development shell (provides rust toolchain)
nix develop

# Enter ARMv7 release cross-compilation shell
nix develop .#armv7-glibc-release

# Enter ARMv7 debug cross-compilation shell
nix develop .#armv7-glibc-debug
```

### Deploying to Device

Deploying to, inspecting, and iterating on a real Deck — the `nix run .#deck` harness (`init`/`deploy`/`sysupgrade`),
the `nix-cargo-deploy.sh` fast path, and on-device log/cache/config — is documented in
**[`docs/deployment.md`](docs/deployment.md) ("Deck Device Operations")**.

### Cargo Commands

Standard cargo commands work within the nix development shells:

```bash
# Check for compilation errors
cargo check

# Run all tests
cargo test

# Run clippy lints (matches CI: workspace lints from Cargo.toml + tests, all warnings are errors)
cargo clippy --workspace --tests -- -D warnings

# Format code
nix fmt

# Build (use within nix develop shell for correct toolchain)
cargo build
cargo build --release
```

### CI/Nix Checks

We use GitLab, the checks are in `.gitlab-ci.yml`, mostly using flake outputs.

## Code Style and Linting

### Rust


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BraiinsForge/bmc-main](https://github.com/BraiinsForge/bmc-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
