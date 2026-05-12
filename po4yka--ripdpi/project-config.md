---
trigger: always_on
description: RIPDPI is an Android VPN/proxy application for DPI (Deep Packet Inspection) bypass. It runs a local SOCKS5 proxy and VPN tunnel through in-repository Rust native modules.
---

# AGENTS.md -- RIPDPI

## Project

RIPDPI is an Android VPN/proxy application for DPI (Deep Packet Inspection) bypass. It runs a local SOCKS5 proxy and VPN tunnel through in-repository Rust native modules.

## Setup

1. Requirements: JDK 17, Android SDK, Android NDK 29.0.14206865, stable Rust toolchain with Android targets, Android CLI (`android`) from `d.android.com/tools/agents` (agents and CI depend on it; `android docs` must be available)
2. Native build properties are defined in `gradle.properties` -- do not hardcode NDK version, ABI filters, or SDK levels elsewhere
3. The Android build invokes the `ripdpi.android.rust-native` convention plugin from `:core:engine`, which builds the native workspace under `native/rust/`
4. Agent runtime permissions for `android` CLI calls:
   - **Claude Code**: `.claude/settings.local.json` allowlists `Bash(android:*)` -- committed, no per-developer action needed
   - **Codex**: approve the project once via `trust_level = "trusted"` in `~/.codex/config.toml` (Codex prompts on first run). Codex has no per-command allowlist; project trust covers all shell invocations including `android docs` / `android sdk` / `android emulator`.

## Build & Test

```bash
./gradlew assembleDebug              # Debug build (includes native compilation)
./gradlew assembleRelease             # Release build (requires signing env vars)
./gradlew testDebugUnitTest           # Run all unit tests
./gradlew :core:data:testDebugUnitTest  # Run tests for a single module
./gradlew staticAnalysis              # Run detekt + ktlint + Android lint
```

## Project Rules

- **Never extend baselines** (detekt, LoC, lint). Fix the underlying violation -- baselines exist only for legacy debt; do not work around CI or hook enforcement.
- **Non-rooted Android baseline** -- the app must fully function on non-rooted devices. Root-only features (`ripdpi-root-helper`, `FakeRst`, `MultiDisorder`, `IpFrag2`) are opt-in behind the `root_mode_enabled` setting and must degrade gracefully when root is unavailable.
- **No backend server** -- all features work offline and locally. Do not design features that require an API endpoint or remote service. External data uses static files on GitHub or bundled assets; user data never leaves the device unless the user explicitly exports it.
- **Goal-driven execution** -- before implementing, convert each task into verifiable success criteria (test name, metric delta, UI render) and verify each before reporting completion. Ask for clarification when criteria are ambiguous rather than guessing.
- **Surface ambiguity early** -- an undocumented JNI contract, a missing schema migration, an unclear protobuf field number, a `DesyncMode` without documented activation: name it, do not guess.
- **Reproduce before fixing** -- a packet-smoke scenario, a `cargo nextest` test, or a Roborazzi baseline is the artifact you change; the source edit follows.
- Removing custom detekt rules, lint baselines, or other quality gates is out of scope unless explicitly requested.

## Task Board

This repository uses Obsidian Tasks-compatible Markdown task lines as the canonical task system.
Use the `repo-task-board` skill for all task-related operations.

Canonical files:

- `docs/tasks/issues/<slug>.md` — **source of truth** — one note per task/epic (YAML frontmatter + canonical `- [ ]` line + spec)
- `docs/tasks/active.md` — Obsidian Tasks query view (`#status/doing`, `#status/review`)
- `docs/tasks/backlog.md` — Obsidian Tasks query view (`#status/backlog`)
- `docs/tasks/blocked.md` — Obsidian Tasks query view (`#status/blocked`)
- `docs/tasks/epics.md` — Obsidian Tasks query view (`#area/epic`)
- `docs/tasks/dashboard.md` — Obsidian Tasks query hub + Bases view links
- `docs/tasks/board.md` — Kanban board (visual layer; source of truth is `issues/`)

Canonical task syntax (lives inside `docs/tasks/issues/<slug>.md`):

```md
- [ ] #task <imperative title> #repo/RIPDPI #area/<area> #status/<status> <priority>
```

Per-task note YAML frontmatter:

```yaml
---
title: Imperative task title
type: task            # task | epic
status: doing         # backlog | todo | doing | review | blocked | done | dropped
area: diagnostics     # engine | rust-native | diagnostics | transport | outbound | dns |
                      # routing | vpn | proxy | relay | android | ui | data | service |
                      # testing | ci | epic
priority: high        # critical | high | medium | low
owner: Role name
parent: epic-slug     # slug of parent epic, or null
blocks: []
blocked_by: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Lifecycle: create via Templater template → transitions update `status:` + `#status/*` tag → delete file on close (git history is the audit trail). Do NOT add task lines to `active.md`, `backlog.md`, `blocked.md`, or `epics.md` — those are query-only views.

Invoke the `repo-task-board` skill when the user mentions: roadmap, TODO, backlog, Kanban, task board, sprint, blocked work, or agent-ready work.

## Architecture

```
:app (UI/Compose) --> :core:service (VPN/proxy services)
                          |
                     :core:engine (Rust native + JNI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [po4yka/RIPDPI](https://github.com/po4yka/RIPDPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
