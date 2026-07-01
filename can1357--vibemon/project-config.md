---
trigger: always_on
description: Vibemon (`vmon`) is a small KVM/HVF-based Linux microVM monitor. It pairs a Rust VMM core with a Python orchestration layer that offers a Docker-like CLI, a Modal-style sandbox SDK, a daemon, a REST/WebSocket server, and a React web panel.
---

# Repository Guidelines

## Project Overview

Vibemon (`vmon`) is a small KVM/HVF-based Linux microVM monitor. It pairs a Rust VMM core with a Python orchestration layer that offers a Docker-like CLI, a Modal-style sandbox SDK, a daemon, a REST/WebSocket server, and a React web panel.

- **Platforms:** Linux + KVM (`x86_64`, `aarch64`); macOS 15+ Apple Silicon + HVF (`aarch64` guests only). Backend is selected at compile time — there is no runtime switch, and `x86_64` macOS is unsupported.
- **Capabilities:** direct-kernel and UEFI boot, virtio devices (blk/net/console/fs), snapshot / restore / fork with copy-on-write, in-process sandboxing (seccomp + Landlock + jailer), warm pools, secrets, volumes, egress control, PTY exec, and metrics.

## Architecture & Data Flow

Three layers. The Python daemon spawns one Rust `vmm` process per microVM; the guest agent runs inside the VM and talks back over a virtio-console channel.

```
Web UI (React SPA)
   │ HTTP / WebSocket
vmon serve (FastAPI, server.py)
   │ Unix socket  $VMON_HOME/vmond.sock
vmond (daemon.py) ──> Engine (core.py, single registry owner)
   │ spawns subprocess per VM, --api-sock JSON control socket
vmm binary (Rust VMM)
   │ virtio-console, length-prefixed binary frames (GC4 / proto.rs)
vmon-agent (guest agent, Linux guest only)
```

**Rust boot path:** `Config::from_args()` → `vmm::run()` → `Vmm::build()` (boot or restore/fork) → allocate guest memory, instantiate virtio device backends, register on the device `Bus` → `Vmm::start()` spawns one thread per vCPU and one worker thread per device. vCPU threads run the hypervisor loop (`KVM_RUN` / HVF), trap MMIO/PortIO to the `Bus`, and notify virtio queues; device workers `poll()` queue/backend/control eventfds and signal completion interrupts.

**Control plane:** Unix-socket JSON protocol (`ping`, `info`, `pause`, `resume`, `snapshot`, `quit`, `metrics`, `extend`). The socket thread never touches the `Vmm` directly — requests cross a `flume` channel to the owner thread. `PauseGate` quiesces vCPUs via an RT signal without `SA_RESTART` on Linux (handler is a no-op; `EINTR` rechecks run state) and via a backend kicker callback on HVF.

## Key Directories

- `src/` — Rust VMM core (the `vmm` binary).
  - `src/hv/` — hypervisor seam; `kvm/` and `hvf/` backends selected by `#[cfg(target_os)]`.
  - `src/arch/` — architecture-specific boot/setup (`x86_64/`: MP table, GDT, MSR; `aarch64/`: FDT, GIC).
  - `src/virtio/` — virtio device model: `mod.rs` (trait + worker loop), `mmio.rs`, `pci.rs` (x86_64-only), `net.rs`, `block.rs`, `fs.rs`, `console.rs`.
  - `src/os/` — OS primitives (`EventFd`: real `eventfd(2)` on Linux, pipe-backed shim on macOS).
  - `src/devices/`, `src/snapshot/`.
- `agent/` — `vmon-agent` guest agent crate (Linux guest only).
- `tests/` — Rust integration tests; shared helpers in `tests/common/mod.rs`.
- `python/vmon/` — Python package (CLI, daemon, server, Engine, sandbox SDK). Bundled assets: `_agent/` (static guest agent), `web/` (built UI).
- `python/tests/`, `python/e2e.py`, `python/cli_e2e.py` — Python unit + e2e suites.
- `ui/` — React + Vite + TypeScript web panel; **builds into `python/vmon/web/`**.
- `demo/` — runnable demo and asset-fetch scripts (Ubuntu/arm64 boots, OCI→ext4, Lima bridge for macOS).

## Development Commands

`just` is the canonical task runner. Recipes are OS-conditional (Linux uses `sudo` for `/dev/kvm` + TAP; macOS auto-codesigns).

```bash
just build           # debug build (auto-codesigns on macOS)
just release         # release build (+ codesign on macOS)
just run *args       # build then run vmon (sudo on Linux)
just format          # format every language (biome | ruff | cargo fmt)
just lint            # lint every language (biome | ruff | clippy)
just check           # type-check every language (tsc | mypy | cargo check)
just test            # cargo test (unit + integration; KVM-gated cases auto-skip)
just integration     # VMON_E2E=1 cargo test --tests -- --test-threads=1
just soak            # VMON_E2E=1 VMON_SOAK=1 cargo test --test soak -- --test-threads=1
just fetch-assets    # ./demo/fetch-test-assets.sh  (kernels/images → target/test-assets/)
just ui              # cd ui && bun install && bun run build  → python/vmon/web/
just agent-musl      # build static vmon-agent → python/vmon/_agent/vmon-agent-<arch>
```

macOS HVF requires the `vmm` binary to be ad-hoc codesigned with `hvf.entitlements` (`com.apple.security.hypervisor`) before running — `just codesign` / `just build` handle this. Hypervisor.framework needs no root; only vmnet networking needs `sudo`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [can1357/vibemon](https://github.com/can1357/vibemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
