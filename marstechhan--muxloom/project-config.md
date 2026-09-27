---
trigger: always_on
description: These rules define the product invariants that changes in this repository must
---

# Muxloom Engineering Guide

These rules define the product invariants that changes in this repository must
preserve.

## Runtime boundaries

- `muxloom` is the controller. `muxloomd` owns remote PTYs, session metadata,
  append-only history, and file operations. Closing the controller or losing an
  SSH connection must not terminate a running child process.
- The word is wrong and stays for now. `muxloom` watches and relays; it does not
  control anything, and more than one may watch a daemon at once, so the pair is
  really viewer and daemon. Say viewer in new prose where it costs nothing.
  Renaming the existing 696 sites is deferred: the `controller` Cargo feature is
  in `default`, so it cannot move without breaking build commands outside this
  repo, and split naming reads worse than the old name used consistently. Do it
  in one sweep, deliberately, or not at all.
- The normal daemon data plane must not depend on target-side `tmux`, `file`,
  `ffmpeg`, or similar utilities. Transfer encoded media to the controller and
  decode it there; never stream remote RGB frames.
- Initial bootstrap may require SSH and a POSIX shell. When a compatible
  companion is installed, normal operation must use the Rust implementation.
- The explicit tmux path is a compatibility fallback. It must remain usable for
  older sessions, but it must never be selected silently.
- Terminal sessions are ephemeral: removing one deletes it. Supported Codex and
  Claude sessions can be archived, searched, and resumed.
- A temporary session is a scratch pad and gets a scratch folder: the daemon
  makes one it owns, runs the session there whatever directory the client named,
  and removes it when the session ends, is deleted, or is found stale. It must
  never inherit a project directory, and it must never aim the machine's next
  ordinary launch.

## Transport and compatibility

- Keep one persistent SSH bridge as the normal data plane for each target.
  Requests, file streams, status, and reverse-tunnel traffic should multiplex
  through it instead of opening a connection per operation.
- Bootstrap, legacy runtime staging, and compatibility fallbacks may currently
  reuse an SSH ControlMaster and `scp`. Treat these as compatibility paths to be
  converged, not as proof that every target has exactly one SSH process today.
- A new controller must preserve each session kind's supported discovery,
  attach, archive, search, and identification semantics for sessions created by
  older `muxloomd` generations and by the explicit tmux fallback.
- Prefer additive protocol and capability changes. Do not bump the wire
  protocol merely to add a file type, metadata field, or optional feature that
  the controller can normalize safely.
- A compatibility fallback must be visible in the TUI, debug log, and terminal
  notifications. Include the reason and affected machine.
- Compare companion build fingerprints, not only the wire protocol. Fingerprint
  calculation must live in the Rust binaries and must not depend on target
  utilities such as `sha256sum` or `shasum`.
- Provisioning a runtime or a companion onto a remote target must offer the
  target its own download first and fall back to pushing bytes over the
  existing connection. The controller resolves only release metadata; the
  target verifies the payload against that digest before installing it, and a
  companion pull is offered only when the published digest equals the asset we
  would otherwise send, so a pull can never install different bytes than a push.
- A publisher's manifest says what to download, not where from: a payload URL
  that leaves the host the manifest was itself read from must be refused rather
  than followed. Which digest a publisher stands behind is its call, not ours,
  so the algorithm travels with the release and the controller and the target
  must check the same one.
- Every target-side fetch must be bounded — connect timeout, total timeout, and
  a stall guard — so a machine with no route to the release fails in seconds
  instead of hanging the install. When every built-in path fails, the reported
  error must name each attempt rather than only the last one.

## Session keepers and non-disruptive daemon upgrades

- Every managed session is owned by its keeper process: PTY, child process,
  and raw history append, nothing more. The keeper's socket protocol is
  version 1 forever — running keepers outlive arbitrarily many daemon
  generations, so every future daemon must keep speaking it. Do not extend the
  keeper's responsibilities; new behavior belongs in the daemon.
- Controller exit, binary deployment, daemon upgrades, and daemon crashes must
  not stop running agents. Typed input from the daemon goes through the keeper;
  the daemon never owns a session PTY directly.
- Deploy a new binary atomically, then drain the old daemon. Handover requires
  a sole client but not idle sessions: live sessions ride their keepers across,
  and the next generation adopts every keeper socket it finds, rebuilding
  screen and activity state from the history tail.
- Current daemons must enter draining atomically with client registration and
  agent launch. Once draining starts, reject new work, acknowledge handover,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarsTechHAN/Muxloom](https://github.com/MarsTechHAN/Muxloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
