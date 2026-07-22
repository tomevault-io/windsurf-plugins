---
trigger: always_on
description: Use when the spawned workload should outlive the immediate UI action and be supervised by `sp up`.
---

This project is a namespace-based containerization/runtime system with persistent profile state rooted at `/nsp3` by default, overridable per process via `state_paths::set_persist_root`.

Searchable references:
- `PERSIST_ROOT`, `state_paths::persist_root`, `state_paths::set_persist_root` in `crates/common/src/lib.rs`
- `Cli.root` in `crates/nsproxy-core/src/lib.rs`

## Core runtime model

- CLI commands are centralized in `MainCommand`, including `Up`, `Serve`, and `Sandbox`.
  - Search: `enum MainCommand`, `MainCommand::Up`, `MainCommand::Serve`, `MainCommand::Sandbox`
  - File: `crates/nsproxy-core/src/lib.rs`

- Per-profile namespace/process metadata uses `NsAlive` (`child_pid`, `up_pid`, `serve_pid`, `bind_mount`, optional browser profile).
  - Search: `struct NsAlive`
  - Files: `crates/common/src/lib.rs`, usages in `crates/nsproxy-core/src/cmd_common.rs`

- `sp up` daemon exposes `/nsp3/{name}/up.sock` and exchanges `DaemonRequest`/`DaemonEvent` snapshots.
  - Search: `up_sock_path`, `enum DaemonRequest`, `enum DaemonEvent`, `run_up_daemon`, `handle_up_client`
  - Files: `crates/diag/src/lib.rs`, `crates/nsproxy-core/src/bin/nsproxy.rs`

- Structured CLI spawning uses memfd/fd handoff (`sp <fd>` style).
  - Search: `DaemonRequest::SpawnCli`, `CliDaemonRequest::SpawnCli`, `cli_to_memfd`, `cli_to_inheritable_fd`, `spawn_cli_process`
  - Files: `crates/diag/src/lib.rs`, `crates/nsproxy-core/src/lib.rs`, `crates/nsproxy-core/src/bin/nsproxy.rs`

## State + config model

- Persisted state shape is standardized by `PersistentState`; path helpers are in `state_paths`.
  - Search: `trait PersistentState`, `mod state_paths`, `profile_ns_meta`
  - Files: `crates/nsproxy-core/src/state_blueprint.rs`, `crates/common/src/lib.rs`

- `HotConfig` carries DNS/TUN/dev/mount/locals/daemon fields; `TemplateConfig` carries sandbox mode, mounts, chmod, env, hot-config linkage.
  - Search: `struct HotConfig`, `struct TemplateConfig`
  - File: `crates/nsproxy-core/src/lib.rs`

- Path expansion is stateful via `PathExpansionState`.
  - Search: `struct PathExpansionState`, `expand_with`, `expand_source`, `expand_target`
  - File: `crates/nsproxy-core/src/lib.rs`

- Domain normalization helper uses canonical trailing-dot behavior.
  - Search: `normalize_domain`
  - File: `crates/common/src/lib.rs`

## Sandboxing + mounts

- `SandboxMode` models pivot/overlay semantics.
  - Search: `enum SandboxMode`, `TemplateConfig.sandbox_mode`
  - File: `crates/nsproxy-core/src/lib.rs`

- Pivot flow is implemented in `sandbox.rs`.
  - Search: `apply_pivot`, `build_skeleton`, `apply_mounts`, `apply_chmod`, `detect_sandbox_state`, `assert_mount_ns_isolated`
  - File: `crates/nsproxy-core/src/sandbox.rs`

- Mount and pivot primitives are in `sys.rs`.
  - Search: `mount_bind_rw_explicit`, `mount_bind_ro_explicit`, `mount_tmpfs`, `pivot_root_into`, `mount_bind_root`
  - File: `crates/nsproxy-core/src/sys.rs`

- Pivot staging dir helper is profile-scoped.
  - Search: `state_paths::pivot_root_dir`
  - File: `crates/common/src/lib.rs`

## Networking + diagnostics

- `UplinkHub` owns proxy map and pluggable routing function.
  - Search: `struct UplinkHub`, `with_routing`, `set_routing`
  - File: `crates/nsproxy-core/src/uplink.rs`

- Diagnostic sockets:
  - `diag_sock_path` => `/nsp3/{name}/tun_diag.sock`
  - `up_sock_path` => `/nsp3/{name}/up.sock`
  - Search: `diag_sock_path`, `up_sock_path`
  - File: `crates/diag/src/lib.rs`

- Frame protocol is bincode + u32 little-endian length prefix.
  - Search: `encode_frame`, `read_frame`, `write_bincode_frame`, `read_bincode_frame`, `write_bincode_frame_async`, `read_bincode_frame_async`
  - Files: `crates/diag/src/lib.rs`, `crates/nsproxy-core/src/bin/nsproxy.rs`

## Diag protocol codepaths (symbol index)

### Server-side (`sp serve` path)

- Entry path and diag probe:
  - Search: `cmd_serve`
  - File: `crates/nsproxy-core/src/bin/nsproxy.rs`

- Diag server startup and command rx wiring:
  - Search: `Router::init_diag`, `DiagServer::start`, `Router::take_cmd_rx`
  - Files: `crates/nsproxy-core/src/uplink/router.rs`, `crates/diag/src/lib.rs`

- Per-client stream handling:
  - Search: `serve_client`, `DiagServer::emit`, `DiagServer::install_as_global`
  - File: `crates/diag/src/lib.rs`

### Server-side (`sp up` path)

- Up daemon listener:
  - Search: `run_up_daemon`, `diag::init_up_log_broadcast`
  - File: `crates/nsproxy-core/src/bin/nsproxy.rs`

- Per-client request/event loop:
  - Search: `handle_up_client`, `DaemonRequest::GetProcessList`, `DaemonRequest::Spawn`, `DaemonRequest::SpawnCli`, `DaemonRequest::Stop`
  - File: `crates/nsproxy-core/src/bin/nsproxy.rs`

- One-shot sync path (CLI -> up.sock):
  - Search: `write_bincode_frame`, `read_bincode_frame`
  - File: `crates/nsproxy-core/src/bin/nsproxy.rs`

### Client-side (UI supervisor)

- Up/diag client spawn guards:
  - Search: `ensure_up_client`, `ensure_diag_client`
  - File: `crates/nsproxy-ui/src/supervisor.rs`

- Connected stream loops:
  - Search: `up_stream_loop`, `diag_stream_loop`
  - File: `crates/nsproxy-ui/src/supervisor.rs`

- Retry loops and backoff:
  - Search: `up_client_loop`, `diag_client_loop`, `retry_delay`, `sleep_or_cancelled`
  - File: `crates/nsproxy-ui/src/supervisor.rs`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ple1n/nsproxy](https://github.com/ple1n/nsproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
