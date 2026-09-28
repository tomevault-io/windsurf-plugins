---
trigger: always_on
description: > `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly — see [update-agent-files](vibe/skills/update-agent-files.md).
---

# Mikey — Agent Instructions

> `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly — see [update-agent-files](vibe/skills/update-agent-files.md).

## The Brain (Source of Truth)

The brain in [`vibe/`](vibe/index.md) is the absolute source of truth for every architectural and implementation decision. It wins over this file on any conflict. Read the index before writing any code:

@vibe/index.md

| Part | Folder | Purpose |
|---|---|---|
| **Core** | [`vibe/core/`](vibe/core/) | Vision, principles, scope, UX specs, roadmap, risks, decisions |
| **Architecture** | [`vibe/architecture/`](vibe/architecture/) | System design, connection levels, trust, wire protocol, media, tech stack, security, repo layout |
| **Rules** | [`vibe/rules/`](vibe/rules/) | Coding rules, design language, performance budgets, test matrix, agent files |
| **Skills** | [`vibe/skills/`](vibe/skills/) | Playbooks: build & run Android, add dependency, change protocol, change decision, update agent files |

---

## Project Overview & Ownership

- **Project in one line**: Android app (Kotlin/Compose) + per-user PC tray app (Rust, binary `mikey`) that turns a phone into a mic and webcam for a PC over four automatic connection levels:
  `1 USB debugging (adb reverse)` > `2 USB tethering` > `3 Bluetooth RFCOMM (audio only)` > `4 Wi-Fi`.
- **Ownership boundaries**:
  - `android/` — repo owner's domain.
  - `pc/` — separate contributor's domain. **Do not modify or add code in `pc/` unless explicitly requested by the user.**
- **Roles**: The phone is **always the client**; the PC is **always the server**.

---

## Architecture & Invariants

1. **Android App** (`android/`): `MikeyService` foreground service owns capture + transport. `TransportManager` negotiates the best of 4 levels with make-before-break upgrades.
2. **PC Tray App** (`pc/`): TCP `:7653`, UDP discovery beacon `:7654`, RFCOMM server, adb watcher, `SessionManager` (tokens, trust, ask-before-join), audio pipeline (Opus, jitter buffer, drift resampler, noise gate, RNNoise, SpeexDSP AEC), video pipeline (JPEG -> virtual camera).
3. **Threading & Concurrency**:
   - PC: Blocking std threads + bounded channels.
   - **The ONLY async code allowed is Tokio inside `pc/src/transport/bt.rs` on Linux** (required by `bluer`). Everywhere else, Tokio and async runtime are strictly forbidden.
4. **Real-time Safety**:
   - Audio capture and playback paths must never block on I/O or locks held by other threads. Use bounded, lock-free, or try-lock handoffs; when full, drop oldest data.
   - Every socket must have a timeout. Every queue/channel must have a strict upper bound. No unbounded buffering.
5. **State & Permissions**:
   - Mic and camera always initialize in the **OFF** state. Never auto-enable capture from the background or upon connection.
   - UI never owns logic: Compose observes `StateFlow` from `MikeyService`; PC tray reflects `SessionManager`.

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
   - Do not add any new library or crate unless listed in [`vibe/architecture/tech-stack.md`](vibe/architecture/tech-stack.md).
   - Any dependency addition requires explicit user approval and following [`add-a-dependency`](vibe/skills/add-a-dependency.md).
5. **No Code in Documentation**:
   - Never place operational code inside `vibe/`. Notes and design thoughts belong in discussions or edits to the brain.

---

## Git Workflow: Branching, Staging & Commits

Strict Git discipline must be maintained at all times.

### 1. Branching Rules
- **Naming format**: `<type>/<short-kebab-description>`
  - `feat/pc-audio-receiver`
  - `fix/pc-cross-platform-and-protocol`
  - `chore/update-dependencies`
  - `docs/clarify-wire-spec`
- **Scope**: Keep branches short-lived and focused on a single feature, fix, or task.
- **Base**: Always branch from and target `main` (or the active tracking branch specified by the user).
- **Protected branches**: Never commit directly to `main` without explicit user direction.

### 2. Staging Rules
- **No blind mass-staging**: **NEVER run `git add .` or `git add -A`**.
- **Inspect before staging**: Always run `git status` and `git diff` to review modified files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diveshpatil9104/mikey](https://github.com/diveshpatil9104/mikey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
