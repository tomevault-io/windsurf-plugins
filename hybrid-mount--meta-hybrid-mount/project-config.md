---
trigger: always_on
description: Guidance for AI coding agents working in this repo. Read this **before** editing or running anything. For user-facing feature docs, see `../README.md` (also `README_ZH.md` / `README_JP.md`). This file covers the things that are easy to get wrong.
---

# AGENTS.md

Guidance for AI coding agents working in this repo. Read this **before** editing or running anything. For user-facing feature docs, see `../README.md` (also `README_ZH.md` / `README_JP.md`). This file covers the things that are easy to get wrong.

## What this project is

**Hybrid Mount** (`hybrid-mount`) is a **mount-orchestration metamodule for KernelSU / APatch** (Android root frameworks). It runs as **root, during boot**, and merges other modules' files into read-only Android system partitions using one of three backends:

- **OverlayFS** — layered mounts (default).
- **Magic Mount** — bind mounts; also the fallback when OverlayFS is unavailable.
- **Kasumi** — an LKM-backed backend with runtime hide/spoof/stealth features.

It is a **Rust binary** (`src/`) plus a **SolidJS WebUI** (`webui/`), packaged with shell scripts (`module/`) into a flashable ZIP by the build tool (`xtask/`).

## Hard constraints — read first

1. **This only builds for `aarch64-linux-android`.** There is no host-platform build. Building/running on Windows or x86 Linux is **not** a supported workflow.
   - The vast majority of real logic is gated behind `#[cfg(any(target_os = "linux", target_os = "android"))]` and calls Linux mount syscalls (`rustix`, `libc`, `procfs`, `loopdev`, `ksu`). It cannot execute on a normal dev host.
   - Building requires: **Rust nightly** (pinned by `rust-toolchain.toml`), **Android NDK r27+**, and **`cargo-ndk`**. WebUI needs **Node 20+** and **pnpm**.
   - **Do not assume you can `cargo check`/`cargo run` to validate.** On a host without the NDK/Android target it will fail. Validate by reading carefully + relying on CI, or build inside the proper NDK environment. State explicitly when you could not build/test.

2. **You cannot meaningfully run the module.** It mounts partitions on a booted, rooted Android device. There is no local "run the app" loop. The pure-logic unit tests (e.g. `src/domain/mod.rs`, `xtask`) are the only thing runnable off-device.

## Feature flags — the #1 thing agents break

There are **three build flavors**, expressed as Cargo feature combinations. **Any change must keep all three compiling.**

| Flavor | Cargo features | What's included |
|--------|----------------|-----------------|
| **full** (default) | `kasumi` (implies `control-plane`) | All 3 backends, WebUI, daemon, CLI, Kasumi LKM |
| **lite** | `control-plane` only (`--no-default-features --features control-plane`) | WebUI + daemon + CLI + OverlayFS/Magic; **no Kasumi** |
| **nano** | none (`--no-default-features`) | Mount-only binary; **no daemon, no CLI, no WebUI, no Kasumi** |

Consequences when editing Rust:

- Code touching Kasumi must be behind `#[cfg(feature = "kasumi")]`.
- Code touching the daemon / CLI / WebUI API must be behind `#[cfg(feature = "control-plane")]`.
- `main.rs` branches: with `control-plane` it parses CLI (`core::entry::run`); without it, it runs `core::startup::run_default()` (mount once and exit).
- When you add a `#[cfg(feature = ...)]` block, make sure the **non-feature path still compiles** (provide the `#[cfg(not(...))]` counterpart where a value is needed — see `core/ops/executor/mod.rs` for the pattern).
- `cargo clippy --all-features` (what `xtask lint` runs) won't catch a flavor that *removes* features. Mentally check the `nano` (no-default-features) build too.

## Architecture / mental model

Boot pipeline (`module/metamount.sh` → the binary → `core::startup`):

```
config.toml ─► Inventory (scan modules) ─► Planner (apply rules) ─► Executor (overlay/magic/kasumi) ─► Finalize (persist state, cleanup)
```

The executor is driven by a **typestate state machine**: `MountController<Init> → StorageReady → Planned → Executed` in `src/core/controller.rs`. Each transition is one pipeline stage.

Source layout (`src/`):

- `conf/` — config schema (`schema.rs`), TOML loader, `clap` CLI definition (`cli.rs`), CLI handlers.
- `domain/` — core enums: `MountMode` (overlay/magic/kasumi/ignore), `ModuleRules`, the **rule-resolution logic** (path override → module default → global default). Has the only substantial unit tests.
- `partitions.rs`, `defs.rs` — managed partition lists and **all the hardcoded `/data/adb/...` paths**. Treat these constants as load-bearing.
- `core/` — `inventory/` (module discovery), `ops/` (plan + per-backend executors), `daemon/` (Unix socket for CLI + TCP/SSE HTTP for WebUI), `api/` (WebUI payload builders), `startup/` (boot + recovery/retry), `storage/` (ext4 image / tmpfs), `runtime_state.rs`, `kasumi_coordinator.rs`.
- `mount/` — the three backends: `overlayfs/`, `magic_mount/`, `kasumi/`, plus `umount_mgr.rs`.
- `sys/` — low-level syscalls: `mount.rs`, `lkm.rs` (LKM load/unload), Kasumi UAPI bindings (generated by `build.rs` via `bindgen` from `src/sys/kasumi_uapi.h`), `nuke.rs`.
- `utils/` — logging, path helpers, validation.

`webui/` is a standalone SolidJS app (Vite, TypeScript, Material Web). `webui/src/lib/constants_gen.ts` is **generated** at build time — don't hand-edit it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hybrid-Mount/meta-hybrid_mount](https://github.com/Hybrid-Mount/meta-hybrid_mount) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
