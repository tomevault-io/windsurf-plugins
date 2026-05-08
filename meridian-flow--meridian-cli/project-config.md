---
trigger: always_on
description: No real users, no real user data. No backwards compatibility needed — completely change the schema to get it right.
---

# Development Guide: meridian-cli

No real users, no real user data. No backwards compatibility needed — completely change the schema to get it right.

(if this is CLAUDE.md, it is symlink to AGENTS.md)

## Philosophy

**Meridian-Channel** is a coordination layer for multi-agent systems — not a file system, execution engine, or data warehouse.

### Design Principles

1. **Separate Policy from Mechanism** *(Raymond, Rule of Separation)*: Harness adapters are mechanism (how to launch Claude/Codex/OpenCode). CLI commands are policy (what to do, which model, what output). Policy changes fast; mechanism stays stable. Keep them apart.
2. **Extend, Don't Modify** *(Open/Closed)*: New harness = one adapter file + registration. New package source = one mars config entry. New CLI command = one module. If a feature requires editing 10 files, the abstraction is wrong.
3. **Knowledge in Data, Not Code** *(Raymond, Rule of Representation)*: Agent capabilities live in YAML profiles, not procedural code. State lives in JSONL events, not in-memory objects. This keeps the system inspectable and harness-agnostic.
4. **Crash-Only Design** *(Candea & Fox)*: Every write is atomic (tmp+rename). Every read tolerates truncation. There is no "graceful shutdown" — if meridian is killed mid-spawn, the next `meridian status` detects and reports the orphaned state. Recovery IS startup.
5. **Progressive Disclosure** *(clig.dev, Lengstorf)*: `meridian spawn "do the thing"` works with smart defaults. Power users override with `--model`, `--harness`, `--skills`. Don't force all-or-nothing configuration.
6. **Simplest Orchestration That Works** *(Google Cloud AI patterns)*: Stay a thin coordination layer. Centralized spawn-and-report is enough. Don't build complex agent choreography until the simple model breaks. "Simplest" means least total complexity owned — lines maintained, platforms tested, failure modes debugged, contributors onboarded — not fewest imports. A trusted library that deletes a subsystem is a simplification; a hand-rolled reimplementation of the same subsystem is not.

### Core Principles

1. **Harness-Agnostic**: One CLI, many runtimes. Meridian never assumes Claude, Codex, or any specific harness — adapters bridge the gap.
2. **Files as Authority**: All state is files — project-level under `.meridian/`, user-level under `~/.meridian/` (see `get_user_home()`). No databases, no services, no hidden state. If it's not on disk, it doesn't exist.
3. **Coordination, Not Control**: Meridian provides structure (spawns, sessions, skills, sync) but never dictates how agents do their work.
4. **Observable by Default, Intrusive Only Where Observation Requires It**: Meridian reads harness state rather than driving harness behavior. Where observation needs a mechanism the harness doesn't provide — like capturing output from a TUI that only emits to a TTY — meridian reaches into the boundary with the minimum machinery needed (PTY capture for primary-launch session-ID extraction is the canonical example). Observability requirement, not control lever. Code that looks intrusive should be justified against a specific unobservable-otherwise constraint, and that constraint named in the commit or comment.
5. **Idempotent Operations**: `meridian sync` twice = same result. Re-running after a crash converges to correct state, never doubles side effects.
6. **Windows Is First-Class**: Windows support is a product requirement, not cleanup work. Do not ship path logic, process behavior, filesystem assumptions, or tests that only work on POSIX unless the limitation is explicitly accepted and documented. Design root discovery, env handling, locking, signals, shell invocation, and smoke-test coverage with Windows semantics in mind from the start.
7. **Prefer Cross-Platform Abstractions**: In Rust, default to `std` and mature cross-platform crates over handwritten OS-specific branches. Use direct platform-specific APIs only behind narrow adapter boundaries and only when a cross-platform abstraction is insufficient. A dependency that deletes platform-specific code and test matrix burden is a simplification, not bloat.
8. **No VCS Dependency for Core Functionality**: Meridian must work in plain directories without git. Git metadata (remotes, commit history, worktree structure) may be used as optional hints or heuristics, but core operations — project identity, state resolution, spawn coordination, session tracking — must not require a git repository.

### Architecture

- **State Root**: User-level `~/.meridian/` (via `get_user_home()`) is the primary state root — spawns, sessions, work items. Project-level `.meridian/` holds project identity and package config. Migration toward user-level is intentional and ongoing.
- **Harness Adapters**: `src/meridian/lib/harness/` — per-harness command building, output extraction, materialization. Adding a harness = one adapter file + registration.
- **State Layer**: `src/meridian/lib/state/` — path resolution, spawn store, session store. Atomic writes via tmp+rename, `fcntl.flock` for concurrency.
- **Package Sync**: `meridian mars ...` — package resolution and `.agents/` materialization are delegated to mars.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meridian-flow/meridian-cli](https://github.com/meridian-flow/meridian-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
