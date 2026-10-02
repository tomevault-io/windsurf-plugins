---
trigger: always_on
description: > `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly - see [update-agent-files](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).
---

# Owlmic - Agent Instructions

> `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly - see [update-agent-files](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).

## The Source of Truth

The documentation in [`docs/`](docs/README.md) is the absolute, living source of truth for every architectural and implementation decision. It wins over this file on any conflict. Read the index before writing any code:

@docs/README.md

| Part | Document | Purpose |
|---|---|---|
| **Vision & Scope** | [`docs/PRODUCT_VISION_AND_SCOPE.md`](docs/PRODUCT_VISION_AND_SCOPE.md) | Vision, principles, scope, user journeys, roadmap, risks, decisions |
| **Architecture** | [`docs/SYSTEM_ARCHITECTURE.md`](docs/SYSTEM_ARCHITECTURE.md) | System design, connection levels, trust, wire protocol, media pipelines, repo layout |
| **Rules & Budgets** | [`docs/PERFORMANCE_AND_REALTIME_BUDGETS.md`](docs/PERFORMANCE_AND_REALTIME_BUDGETS.md) | Coding rules, design language, performance budgets, test matrix |
| **Playbooks & Skills** | [`docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md`](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md) | Playbooks: build & run Android, add dependency, change protocol, change decision, update agent files |

### Mandatory Documentation Synchronization (Zero Drift)
Code and documentation must **never** drift apart:
- **Always read `docs/` first**: Before proposing or modifying code, inspect the corresponding document in `docs/` to uphold architecture, budgets, and invariants.
- **Update `docs/` on every change - even minute ones**: Whenever **any** change is made to the codebase (including minute details such as protocol opcodes, buffer capacities, timeouts, port numbers, UI copy, thresholds, hysteresis values, or dependency versions), the corresponding document in `docs/` **must be updated in lockstep**.
- **No rogue documentation directories**: All project documentation, specs, and playbooks live exclusively in `docs/`.

---

## Project Overview & Ownership

- **Project in one line**: Android app (Kotlin/Compose) + per-user PC tray app (Rust, binary `owlmic`) that turns a phone into a mic and webcam for a PC over four automatic connection levels:
  `1 USB debugging (adb reverse)` > `2 USB tethering` > `3 Wi-Fi` > `4 Bluetooth RFCOMM (audio only)`.
- **Ownership boundaries**:
  - `android/` - repo owner's domain.
  - `pc/` - separate contributor's domain. **Do not modify or add code in `pc/` unless explicitly requested by the user.**
- **Roles**: The phone is **always the client**; the PC is **always the server**.

---

## Architecture & Invariants

1. **Android App** (`android/`): `OwlmicService` foreground service owns capture + transport. `TransportManager` negotiates the best of 4 levels with make-before-break upgrades.
2. **PC Tray App** (`pc/`): TCP `:7653`, UDP discovery beacon `:7654`, RFCOMM server, adb watcher, `SessionManager` (tokens, trust, ask-before-join), audio pipeline (Opus, jitter buffer, drift resampler, noise gate, RNNoise, SpeexDSP AEC), video pipeline (JPEG -> virtual camera).
3. **Threading & Concurrency**:
   - PC: Blocking std threads + bounded channels.
   - **The ONLY async code allowed is Tokio inside `pc/src/transport/bt.rs` on Linux** (required by `bluer`). Everywhere else, Tokio and async runtime are strictly forbidden.
4. **Real-time Safety**:
   - Audio capture and playback paths must never block on I/O or locks held by other threads. Use bounded, lock-free, or try-lock handoffs; when full, drop oldest data.
   - Every socket must have a timeout. Every queue/channel must have a strict upper bound. No unbounded buffering.
5. **State & Permissions**:
   - Mic and camera always initialize in the **OFF** state. Never auto-enable capture from the background or upon connection.
   - UI never owns logic: Compose observes `StateFlow` from `OwlmicService`; PC tray reflects `SessionManager`.

---

## Anti-AI Slop & Lean Engineering Rules

Write lean, intentional, production-grade code. No AI slop or LLM artifacts:

1. **Build strictly what is asked for the current phase**:
   - Zero speculative future-proofing or "just-in-case" flexibility.
   - No generic helper classes, theoretical extension points, or unused utility functions.
   - No unnecessary design patterns: avoid premature factories, builders, or multi-layer wrappers around single operations.
2. **Zero Code Churn & Surgical Diffs**:
   - Keep edits minimal, precise, and targeted.
   - Never rewrite, reorder, or reformat working code outside the task scope.
   - Never rewrite an entire file when changing a few lines suffices.
   - Respect existing file conventions and indentation.
3. **No Chatty Comments**:
   - Do not write comments that narrate the syntax or state the obvious (e.g., `// loop through items`, `// set timeout to 5 seconds`).
   - Only write comments to explain non-obvious **why**, invariants, or hardware/OS quirks.
4. **Strict Dependency Austerity**:
   - Do not add any new library or crate unless listed in [`docs/SYSTEM_ARCHITECTURE.md`](docs/SYSTEM_ARCHITECTURE.md).
   - Any dependency addition requires explicit user approval and following [`add-a-dependency`](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).
5. **No Code in Documentation**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diveshpatil9104/owlmic](https://github.com/diveshpatil9104/owlmic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
