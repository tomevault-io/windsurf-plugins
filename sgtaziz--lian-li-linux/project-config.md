---
trigger: always_on
description: This file defines how coding agents should work in this repository. It applies to the entire repository unless a more specific `AGENTS.md` exists below the file being changed.
---

# AGENTS.md

This file defines how coding agents should work in this repository. It applies to the entire repository unless a more specific `AGENTS.md` exists below the file being changed.

## Project priorities

Lian Li Linux is a long-running hardware-control daemon with a Tauri desktop client. Correct device behavior, low background resource use, safe shutdown, and compatibility with existing user configuration take priority over convenience or large refactors.

Treat regressions that can wedge hardware, leave fans or pumps uncontrolled, blank displays, saturate USB, consume a CPU core, grow memory without bound, or prevent clean shutdown as release blockers.

If a requested change overlooks a serious correctness, safety, compatibility, or resource issue, point it out clearly. Give concrete evidence and explain the user-visible consequence. Do not silently implement a narrowly requested change when a closely related glaring issue would make the result unsafe or incomplete.

## Non-negotiable rules

### Comments

Prefer clear names, small functions, and straightforward control flow over comments.

Do not add comments unless they are necessary to explain code that remains complex or difficult to understand after reasonable simplification. When a comment is necessary:

- Keep it concise.
- Explain why the code exists or which invariant it preserves.
- Do not narrate what the next line does.
- Use plain text without decorative symbols, diagrams, banners, issue markers, or changelog-style history.
- Place it next to the code it explains so it moves with that code.

When editing a region, remove stale, redundant, overly long, or detached comments you encounter. Preserve comments that document protocol facts, unsafe invariants, hardware timing requirements, or behavior that cannot be made clear through naming alone. Do not perform an unrelated repository-wide comment cleanup unless asked.

Public API documentation should follow the same standard. Add it only when callers need information that the type or function signature cannot express.

### Git and GitHub

Never commit code unless the user explicitly asks for a commit. Never push code unless the user explicitly asks for a push. Never create or update a pull request unless the user explicitly asks for that pull request action.

Do not treat a request to implement, fix, test, review, or prepare changes as permission to commit, push, or create a pull request.

When a commit is explicitly requested, use Conventional Commits with a concise scope naming the affected device or feature:

```text
fix(h2): restore brightness after daemon restart
fix(wireless): bound failed discovery retries
feat(rgb): add per-zone direction control
fix(tl-lcd): validate frame payload size
```

Use an imperative, compact subject. Keep it specific and omit generic subjects such as `fix bug`, `updates`, or `misc changes`. Common scopes include `h2`, `hydroshift`, `tl`, `tl-lcd`, `wireless`, `rgb`, `lcd`, `media`, `daemon`, `gui`, `ipc`, `evdi`, `packaging`, and `docs`.

Use a subject-only commit message, with no body. Use `sgtaziz` for project maintainer attribution.

### Tests

Do not add tests merely to increase coverage or mirror implementation details.

Tests must validate meaningful input and output behavior, including protocol bytes, parsing, serialization, state transitions, boundary conditions, error results, or regressions observable by a caller. A regression test should fail for the reported bug and pass after the fix.

Hardware-independent logic should be separated from device I/O when that produces a useful deterministic test. Do not build elaborate mocks for behavior whose only meaningful validation requires physical hardware. Record required manual hardware checks in the handoff instead.

## Repository map

The active Rust workspace members are:

| Path | Responsibility |
| --- | --- |
| `crates/lianli-shared` | Configuration schema, IPC types, device identifiers, sensor types, RGB and fan models, screen capabilities, and shared templates |
| `crates/lianli-control` | Installation health, native and Distrobox service management, recoverable state transfer, managed media storage, and diagnostic log collection |
| `crates/lianli-transport` | HID and USB transport wrappers, timeouts, retry behavior, and transport errors |
| `crates/lianli-devices` | Device detection, protocol implementations, device traits, wireless control, and per-family drivers |
| `crates/lianli-media` | Image, GIF, video, H.264, sensor, and custom-template rendering |
| `crates/lianli-evdi` | Safe wrapper around EVDI for desktop-mode displays |
| `crates/lianli-display` | Hyprland, Hermes-KMS and EVDI capture backends, GPU buffers, graphical-session discovery, and display channels |
| `crates/lianli-session` | Graphical-session capture helper, capture worker supervision, and desktop stream encoding |
| `crates/lianli-daemon` | Service lifecycle, polling, controllers, media streaming, IPC server, persistence, and shutdown |
| `crates/lianli-gui/src-tauri` | Tauri backend for the desktop client |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sgtaziz/lian-li-linux](https://github.com/sgtaziz/lian-li-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
