---
trigger: always_on
description: VoidBox is a lightweight micro-VM runtime for sandboxed execution. It boots a
---

# AGENTS.md — VoidBox

VoidBox is a lightweight micro-VM runtime for sandboxed execution. It boots a
guest agent from initramfs, optionally performs an OCI root switch, and exposes
a host↔guest API over vsock. This file covers architecture, testing, validation,
and debugging guidance for agents working on the project.

## Code intelligence

**Always prefer LSP operations** (`goToDefinition`, `findReferences`, `hover`,
`documentSymbol`, `workspaceSymbol`, etc.) over Grep/Glob for code navigation.
LSP provides compiler-aware results that understand types, scopes, and cross-file
relationships. Fall back to Grep/Glob only for pattern-based searches LSP doesn't
cover (comments, config files, non-Rust files).

## Rust coding conventions

### Imports and constants

Always declare `use` imports and `const` / `static` items at **module scope**
(top of the file, after the module-level doc comment), never inline inside
function bodies.  Inline declarations hide dependencies and make the file harder
to scan.

```rust
// ✓ correct
use std::io::{Read, Seek, SeekFrom};
const OVERHEAD_BYTES: u64 = 208 * 1024 * 1024;

fn my_fn() { /* uses them */ }

// ✗ avoid
fn my_fn() {
    use std::io::{Read, Seek, SeekFrom};
    const OVERHEAD_BYTES: u64 = 208 * 1024 * 1024;
    // ...
}
```

### Data model and policy design

Prefer **explicit single-source models** over overlapping flags that create
ambiguous or invalid states. If a feature has a small closed set of modes, model
it as an enum rather than separate booleans/options whose combinations must be
interpreted later.

Keep **execution policy** separate from persisted or user-resolved config.
Command-scoped behavior (for example, interactive terminal ownership) should be
derived at the CLI/runtime boundary and passed inward as a narrow policy type,
rather than mutating resolved config structs or threading broad command enums
through lower layers.

Prefer **descriptive local names** and named constants over shorthand locals and
inline literals. Variables like `rc`, `n`, `msg`, `ws`, `k`, and `v` are only
acceptable in tiny, obvious scopes; otherwise use names that describe meaning
directly. Repeated or behavior-defining literals should be lifted to documented
module-scope constants.

### VM pre-flight validation

Operations that can fail silently inside the guest (e.g. the kernel dropping
initramfs files when memory is too low) must surface a clear diagnostic on the
**host**, before the VM starts.  The pattern:

1. **Check before launch** — validate the config in `VmmBackend::start()` before
   any irreversible side effect, using helpers on `BackendConfig`
   (e.g. `initramfs_memory_warning()`).
2. **Log at `WARN`** — use `warn!()` so the message appears without aborting the
   run.  Return `Err` only when a successful boot is impossible regardless of
   retry.
3. **Name the downstream symptom** — mention the observable failure mode
   (e.g. *"kernel may silently drop initramfs files (e.g. vsock.ko)"*) so an
   operator can connect the warning to a later guest error without a separate
   debugging session.

## Platform parity

**Contributions must work on both Linux (KVM) and macOS (VZ).** Validate on both
where applicable. Key platform differences:

| Concern | Linux (KVM) | macOS (VZ) |
|---------|-------------|------------|
| Kernel | `vmlinuz` (compressed OK) | `vmlinux` (uncompressed); `download_kernel.sh` uses `extract-vmlinux` |
| Network detection | `virtio_mmio.device=512@0xd0000000:10` in cmdline | `voidbox.network=1` in cmdline (VZ uses PCI) |
| Kernel modules | Host or pinned; `build_test_image.sh` | `guest_macos.sh`; `VOID_BOX_KMOD_VERSION` must match `download_kernel.sh` |

## Architecture overview

- Host runtime launches a VM backend (`KvmBackend` on Linux, `VzBackend` on macOS).
- Guest boots `guest-agent` from initramfs.
- If `sandbox.image` is set, guest performs OCI root switch with `pivot_root`.
- On Linux/KVM, OCI base rootfs is attached as cached `virtio-blk` disk.
- OCI skills are mounted as read-only tool roots.
- If `agent.mode` is `service`, the agent runs indefinitely; output is published while the process continues.
- If `agent.messaging.enabled` is `true`, a host-side sidecar coordinates inter-agent communication over HTTP; `void-mcp` bridges Claude Code to the sidecar via MCP tools.

## Control channel I/O model

The control channel (`src/backend/control_channel.rs`) uses **synchronous I/O**
(`std::io::Read`/`Write`) for all guest communication, wrapped in
`tokio::task::spawn_blocking` so that blocking syscalls never stall the tokio
async runtime.

### Why not fully async (`AsyncRead`/`AsyncWrite`)?

1. **The VZ connector is fundamentally callback-based.**
   `VZVirtioSocketDevice.connectToPort:completionHandler:` dispatches onto a GCD
   serial queue and fires an ObjC completion handler.  There is no fd to poll
   until *after* the callback delivers one.  A bridge channel between GCD and
   tokio is required regardless.

2. **`AsyncFd` on raw vsock fds is fragile.**
   The fd comes from `dup(connection.fileDescriptor())` inside a GCD callback,
   outside tokio's control.  `AsyncFd` requires `O_NONBLOCK`, which changes
   read/write semantics and demands `WouldBlock` handling.  The current blocking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-void-ia/void-box](https://github.com/the-void-ia/void-box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
