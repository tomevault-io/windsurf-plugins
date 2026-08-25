---
trigger: always_on
description: These instructions apply to the whole repository. The active architecture decision baseline for remote work is `zeus/REMOTE_PORT.md`; read it before changing remote session behavior, SSH handling, PTYs, holders, terminal state, or packaging.
---

# Zeus repository instructions

## Scope

These instructions apply to the whole repository. The active architecture decision baseline for remote work is `zeus/REMOTE_PORT.md`; read it before changing remote session behavior, SSH handling, PTYs, holders, terminal state, or packaging.

## Completed remote architecture baseline

- The remote refactor is complete. Maintain the bootstrapped Remote PTY Holder as Zeus's only remote session transport; future remote work extends this architecture rather than reopening the transport migration.
- Implement and maintain remote behavior entirely in the Rust workspace under `zeus/`.
- The product is Rust-only. Do not reintroduce Swift sources, a Swift daemon, or a Swift CLI.
- `zeus/crates/zeus-engine/manifests` is the Agent catalog. A missing manifest does not error at runtime — it silently downgrades that agent to a bare login shell. Never shrink the catalog.
- Existing Rust behavior is the implementation baseline. Historical comments about Swift compatibility do not create new requirements.
- The former Rust SSH + `tmux` transport has been deleted. Never reintroduce it as `legacy_tmux`, a feature flag, a migration path, or a runtime fallback. Missing, corrupt, unsupported, or capability-incompatible Helper artifacts must fail closed with a structured error; an unavailable packaged transport reports `remote_transport_unavailable`.
- When implementation and `zeus/REMOTE_PORT.md` disagree, stop and resolve the design mismatch explicitly instead of silently choosing one.

## Rust workspace map

- `zeus/crates/zeus-engine`: authoritative local session engine, PTY/holder lifecycle, status reduction, host orchestration, and remote bootstrap/SSH seam.
- `zeus/crates/zeus-proto`: shared Rust data models and wire codecs. `remote_pty` is the authoritative versioned Remote Helper protocol; companion access is not part of the current remote transport.
- `zeus/crates/zeus-client`: local app-to-engine client. It should not execute SSH directly.
- `zeus/crates/zeus-term`: GPUI terminal renderer and client-side terminal interaction.
- `zeus/crates/zeus-app`: desktop UI. It requests remote actions through the local Engine.
- `zeus/crates/zeus-cli`: shipped `zeus` automation CLI (hooks, notify, MCP backend, session/worktree commands).
- `zeus/crates/zeus-node`: optional enhanced node mode. It is not a dependency of the default SSH bootstrap path.
- `zeus/crates/zeus-terminal-state`: shared headless terminal parser/Grid/Snapshot/Diff implementation used by the local Engine and remote Holder.
- `zeus/crates/zeus-remote`: minimal remote Helper binary. Keep it independent of GPUI, `zeus-app`, `zeus-client`, and `zeus-node`.

The Rust toolchain is pinned by `zeus/rust-toolchain.toml` to Rust 1.95.0, edition 2024.

## Remote architecture invariants

- SSH is the authenticated encrypted byte transport. Use `ssh -T` for Helper protocol channels; SSH must not own the Agent PTY.
- The current baseline must not require remote `tmux`, `screen`, `zellij`, Node.js, Python, `socat`, `nc`, `curl`, `wget`, or a preinstalled Zeus service.
- Reuse OpenSSH configuration and a finite-lived ControlMaster for performance only. A ControlMaster must never be required for session survival.
- Bootstrap is idempotent: probe platform, select an exact local artifact, upload to a nonce temp path, verify, then atomically rename. Versioned binaries coexist by protocol and Build ID.
- Never overwrite a live Helper version in place. GC must retain every Build ID referenced by a session.
- Never construct Agent launches by concatenating shell strings. Send structured `argv`, `cwd`, and environment over the protocol and exec the argv directly inside the remote PTY child.
- Bootstrap shell commands must be fixed and internally generated. Validate every path component before interpolation and do not follow untrusted symlinks.
- Capture the remote login/cwd environment on the remote host. Do not copy the local process environment wholesale or propagate local secrets and socket paths.
- The remote Holder owns only the PTY, Agent process tree, current terminal grid/modes/cursor, bounded output, exit facts, and controller lease.
- The local Rust Engine owns `SessionRecord`, manifests, status reduction, project/worktree state, GUI events, orchestration, lifecycle policy, and host management.
- Use exactly one independent Holder process and Unix socket per Session; do not add a multi-session Zeus Supervisor to the current baseline.
- A Holder may spawn one minimal liveness guard for its Agent process group. The guard may only wait for Holder pipe closure and kill that one process group; it must not own a PTY, socket, state, or orchestration.
- The app and client must verify the local Engine's explicit Rust identity during `Hello`; fail closed on missing, old, or unknown daemon identities.
- Agent manifests used by the Rust Engine are Rust-owned resources under `zeus-engine`; remote launch must not load Swift resource bundles or fall back to a Swift Holder.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nnayz/zeus](https://github.com/nnayz/zeus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
