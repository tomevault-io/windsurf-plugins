---
trigger: always_on
description: These instructions apply to the whole repository. The active architecture decision baseline for remote work is `diri/REMOTE_PORT.md`; read it before changing remote session behavior, SSH handling, PTYs, holders, terminal state, or packaging.
---

# Diri repository instructions

## Scope

These instructions apply to the whole repository. The active architecture decision baseline for remote work is `diri/REMOTE_PORT.md`; read it before changing remote session behavior, SSH handling, PTYs, holders, terminal state, or packaging.

## Completed remote architecture baseline

- The remote refactor is complete. Maintain the bootstrapped Remote PTY Holder as Diri's only remote session transport; future remote work extends this architecture rather than reopening the transport migration.
- Implement and maintain remote behavior entirely in the Rust workspace under `diri/`.
- Implement and verify every product behavior in the Rust workspace. Historical
  migration documents are context, not an alternate implementation baseline.
- `diri/crates/diri-engine/manifests` is the canonical Agent catalog. Keep at
  least the established 20 manifests and preserve the package-time count gate:
  a missing manifest does not error, it silently spawns a bare login shell.
- The former Rust SSH + `tmux` transport has been deleted. Never reintroduce it as `legacy_tmux`, a feature flag, a migration path, or a runtime fallback. Missing, corrupt, unsupported, or capability-incompatible Helper artifacts must fail closed with a structured error; an unavailable packaged transport reports `remote_transport_unavailable`.
- When implementation and `diri/REMOTE_PORT.md` disagree, stop and resolve the design mismatch explicitly instead of silently choosing one.

## Rust workspace map

- `diri/crates/diri-engine`: authoritative local session engine, PTY/holder lifecycle, status reduction, host orchestration, and remote bootstrap/SSH seam.
- `diri/crates/diri-proto`: shared Rust data models and wire codecs. `remote_pty` is the authoritative versioned Remote Helper protocol; companion access is not part of the current remote transport.
- `diri/crates/diri-client`: local app-to-engine client. It should not execute SSH directly.
- `diri/crates/diri-term`: GPUI terminal renderer and client-side terminal interaction.
- `diri/crates/diri-app`: desktop UI. It requests remote actions through the local Engine.
- `diri/crates/diri-node`: optional enhanced node mode. It is not a dependency of the default SSH bootstrap path.
- `diri/crates/diri-terminal-state`: shared headless terminal parser/Grid/Snapshot/Diff implementation used by the local Engine and remote Holder.
- `diri/crates/diri-remote`: minimal remote Helper binary. Keep it independent of GPUI, `diri-app`, `diri-client`, and `diri-node`.

The Rust toolchain is pinned by `diri/rust-toolchain.toml` to Rust 1.95.0, edition 2024.

## Remote architecture invariants

- SSH is the authenticated encrypted byte transport. Use `ssh -T` for Helper protocol channels; SSH must not own the Agent PTY.
- The current baseline must not require remote `tmux`, `screen`, `zellij`, Node.js, Python, `socat`, `nc`, `curl`, `wget`, or a preinstalled Diri service.
- Reuse OpenSSH configuration and a finite-lived ControlMaster for performance only. A ControlMaster must never be required for session survival.
- Bootstrap is idempotent: probe platform, select an exact local artifact, upload to a nonce temp path, verify, then atomically rename. Versioned binaries coexist by protocol and Build ID.
- Never overwrite a live Helper version in place. GC must retain every Build ID referenced by a session.
- Never construct Agent launches by concatenating shell strings. Send structured `argv`, `cwd`, and environment over the protocol and exec the argv directly inside the remote PTY child.
- Bootstrap shell commands must be fixed and internally generated. Validate every path component before interpolation and do not follow untrusted symlinks.
- Capture the remote login/cwd environment on the remote host. Do not copy the local process environment wholesale or propagate local secrets and socket paths.
- The remote Holder owns only the PTY, Agent process tree, current terminal grid/modes/cursor, bounded output, exit facts, and controller lease.
- The local Rust Engine owns `SessionRecord`, manifests, status reduction, project/worktree state, GUI events, orchestration, lifecycle policy, and host management.
- Use exactly one independent Holder process and Unix socket per Session; do not add a multi-session Diri Supervisor to the current baseline.
- A Holder may spawn one minimal liveness guard for its Agent process group. The guard may only wait for Holder pipe closure and kill that one process group; it must not own a PTY, socket, state, or orchestration.
- The app and client must verify the local Engine's explicit Rust identity during `Hello`; fail closed on missing, old, or unknown daemon identities.
- Agent manifests used by the Rust Engine are Rust-owned resources under `diri-engine`; remote launch must not load another resource bundle or fall back to a different Holder.
- Share terminal parsing through a minimal `diri-terminal-state` crate; do not make `diri-remote` depend on the full Engine or create a second parser implementation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cristicretu/diri](https://github.com/cristicretu/diri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
