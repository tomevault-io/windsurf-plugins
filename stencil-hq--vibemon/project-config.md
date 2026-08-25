---
trigger: always_on
description: Vibemon (`vmon`) is a small KVM/HVF-based Linux microVM monitor. One Rust binary owns the user CLI, `vmon serve` HTTP/WebSocket server (`vmond` crate), and low-level `vmon vmm` per-VM monitor (`vmm` crate). Python and TypeScript are thin client SDKs for the Rust API.
---

# Repository Guidelines

## Project Overview

Vibemon (`vmon`) is a small KVM/HVF-based Linux microVM monitor. One Rust binary owns the user CLI, `vmon serve` HTTP/WebSocket server (`vmond` crate), and low-level `vmon vmm` per-VM monitor (`vmm` crate). Python and TypeScript are thin client SDKs for the Rust API.

- **Platforms:** Linux + KVM (`x86_64`, `aarch64`); macOS 15+ Apple Silicon + HVF (`aarch64` guests only). Backend is selected at compile time — there is no runtime switch, and `x86_64` macOS is unsupported.
- **Capabilities:** direct-kernel and UEFI boot, virtio devices (blk/net/console/fs), snapshot / restore / fork with copy-on-write, in-process sandboxing (seccomp + Landlock + jailer), warm pools, secrets, volumes, lazy S3 mounts, egress control, PTY exec, and metrics.

## Architecture & Data Flow

Three runtime layers. The Rust server owns the registry and spawns one `vmon vmm` child per microVM; the guest agent runs inside the VM and talks back over a virtio-console channel.

```
Web UI / Rust CLI / Python SDK / TypeScript SDK / Go SDK
   │ gRPC (h2c over TCP or UDS); browsers/TS ride a gRPC-over-WebSocket
   │ bridge (`GET /grpc`, proto/vmon/v1/bridge.proto)
vmon serve (Rust axum + tonic, vmond crate)
   │ Engine registry, image pipeline, pools, mesh, volumes
   │ spawns `vmon vmm ... --api-sock <sock>` per VM
vmon vmm (Rust VMM crate)
   │ virtio-console, length-prefixed binary frames (GC4 / proto.rs)
vmon-agent (guest agent, Linux guest only)
```

**Rust boot path:** `Config::from_args()` → `vmm::run()` → `Vmm::build()` (boot or restore/fork) → allocate guest memory, instantiate virtio device backends, register on the device `Bus` → `Vmm::start()` spawns one thread per vCPU and one worker thread per device. vCPU threads run the hypervisor loop (`KVM_RUN` / HVF), trap MMIO/PortIO to the `Bus`, and notify virtio queues; device workers `poll()` queue/backend/control eventfds and signal completion interrupts.

**Control plane:** Unix-socket JSON protocol (`ping`, `info`, `pause`, `resume`, `snapshot`, `quit`, `metrics`, `extend`). The socket thread never touches the `Vmm` directly — requests cross a `flume` channel to the owner thread. `PauseGate` quiesces vCPUs via an RT signal without `SA_RESTART` on Linux (handler is a no-op; `EINTR` rechecks run state) and via a backend kicker callback on HVF.

**Orchestration (v2, `vmond/src/orch/`):** a horizontally-scalable scheduling layer beside the mesh. `vmon sched` servers keep an in-memory worker table fed from Redis (self-expiring `vmon:o:w:<wid>` keys + `vmon:o:workers` stream, followed by a hand-rolled RESP client in `orch/redis.rs`), place creates with power-of-two-choices, and forward directly to the owning `vmon serve` worker's gRPC endpoint — no datastore on the create path. Workers publish batched heartbeats (`orch/worker.rs`) and reject creates with `busy` when full; schedulers penalize and retry elsewhere. Sandbox routes (`vmon:o:sb:<sid>`) are written asynchronously; created/fetched views gain `node` + `endpoint` (the owning worker's URL). A `SET NX PX` leader lease gates the controller janitor (marks sandboxes of dead workers `lost`) and the HPA-like autoscaler (drain keys + `sh -c` scale hooks with `VMON_SCALE_*`/`VMON_DRAIN_WIDS`/`VMON_IDLE_WIDS`). `--redis` omitted embeds the dev-only mini-redis (`orch/miniredis.rs`); the hermetic end-to-end lives in `orch/e2e_tests.rs` (no hypervisor needed).

## Key Directories

- `src/` — Rust top-level `vmon` binary: CLI (`cli.rs`), local/remote tonic transport (`transport.rs`), and context storage (`contexts.rs`).
- `vmm/` — Rust VMM crate used by `vmon vmm`.
  - `vmm/src/hv/` — hypervisor seam; `kvm/` and `hvf/` backends selected by `#[cfg(target_os)]`.
  - `vmm/src/arch/` — architecture-specific boot/setup (`x86_64/`: MP table, GDT, MSR; `aarch64/`: FDT, GIC).
  - `vmm/src/virtio/` — virtio device model: `mod.rs` (trait + worker loop), `mmio.rs`, `pci.rs` (x86_64-only), `net.rs`, `block.rs`, `fs.rs`, `console.rs`.
  - `vmm/src/os/` — OS primitives (`EventFd`: real `eventfd(2)` on Linux, pipe-backed shim on macOS).
  - `vmm/src/devices/`, `vmm/src/snapshot/`.
- `proto/` — `vmon-proto` crate and the API contract: `vmon/v1/api.proto` (five gRPC services; the ONLY API) and `vmon/v1/bridge.proto` (browser WS bridge framing). Rust code generates at build time via protox + tonic; client codegen is checked in.
- `vmond/` — Rust server/engine crate used by `vmon serve`: gRPC services (`api/grpc.rs`), WS bridge (`api/bridge.rs`), remaining HTTP surfaces (healthz, metrics, ports proxy, static UI), registry, image pipeline, mesh, pools, volumes, lazy S3 access (`s3.rs`, `engine/s3proxy.rs`), VM spawn/control, and the v2 orchestration layer (`orch/`: scheduler + worker publisher + controller + autoscaler, served by `vmon sched`).
- `agent/` — `vmon-agent` guest agent crate (Linux guest only).
- `tests/` — Rust integration tests; shared helpers in `tests/common/mod.rs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stencil-hq/vibemon](https://github.com/stencil-hq/vibemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
