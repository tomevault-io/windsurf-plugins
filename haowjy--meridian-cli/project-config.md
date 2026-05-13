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
2. **Separate Concerns, Justify Boundaries** *(Dijkstra, Separation of Concerns)*: Group by concern — things that change together for the same reason belong together. Draw boundaries where concerns actually change independently. A boundary between things that always change together is ceremony, not structure. God-object symptoms are refactor triggers, but splitting into more types that carry the same data is not the fix — independence is.
3. **Extend Through Seams**: New harness = one adapter file + registration. New package source = one mars config entry. New CLI command = one module. New extension/plugin capability = one explicit seam, not scattered edits through core. If a feature requires editing 10 files, the abstraction is wrong.
4. **Knowledge in Data, Not Code** *(Raymond, Rule of Representation)*: Agent capabilities live in YAML profiles, not procedural code. State lives in JSONL events, not in-memory objects. This keeps the system inspectable and harness-agnostic.
5. **Crash-Only Design** *(Candea & Fox)*: Every write is atomic (tmp+rename). Every read tolerates truncation. There is no "graceful shutdown" — if meridian is killed mid-spawn, the next `meridian status` detects and reports the orphaned state. Recovery IS startup.
6. **Progressive Disclosure** *(clig.dev, Lengstorf)*: `meridian spawn "do the thing"` works with smart defaults. Power users override with `--model`, `--harness`, `--skills`. Don't force all-or-nothing configuration.
7. **Simplest Orchestration That Works** *(Google Cloud AI patterns)*: Stay a thin coordination layer. Centralized spawn-and-report is enough. Don't build complex agent choreography until the simple model breaks. "Simplest" means least total complexity owned — lines maintained, platforms tested, failure modes debugged, contributors onboarded — not fewest imports. A trusted library that deletes a subsystem is a simplification; a hand-rolled reimplementation of the same subsystem is not.
8. **Refactor Early, Change Is Cheap** *(Beck, "make the change easy")*: No external users, no real user data, and no backwards-compatibility constraints mean structural debt costs more than fixing it. LLM-driven implementation lowers the execution cost of change; accumulated context and architecture debt raise the reasoning cost. Prefer early cleanup when it reduces future edit fan-out, clarifies ownership, or makes behavior easier to observe and test.

### Core Principles

1. **Harness-Agnostic**: One CLI, many runtimes. Meridian never assumes Claude, Codex, or any specific harness — adapters bridge the gap.
2. **Files as Authority**: All state is files — project-level under `.meridian/`, user-level under `~/.meridian/` (see `get_user_home()`). No databases, no services, no hidden state. If it's not on disk, it doesn't exist.
3. **Coordination, Not Control**: Meridian provides structure (spawns, sessions, skills, sync) but never dictates how agents do their work.
4. **Observable by Default, Intrusive Only Where Observation Requires It**: Meridian reads harness state rather than driving harness behavior. Where observation needs a mechanism the harness doesn't provide — like capturing output from a TUI that only emits to a TTY — meridian reaches into the boundary with the minimum machinery needed (PTY capture for primary-launch session-ID extraction is the canonical example). Observability requirement, not control lever. Code that looks intrusive should be justified against a specific unobservable-otherwise constraint, and that constraint named in the commit or comment.
5. **Idempotent Operations**: `meridian sync` twice = same result. Re-running after a crash converges to correct state, never doubles side effects.
6. **Windows Is First-Class**: Windows support is a product requirement, not cleanup work. Do not ship path logic, process behavior, filesystem assumptions, or tests that only work on POSIX unless the limitation is explicitly accepted and documented. Design root discovery, env handling, locking, signals, shell invocation, and smoke-test coverage with Windows semantics in mind from the start.
7. **Prefer Cross-Platform Abstractions**: In Rust, default to `std` and mature cross-platform crates over handwritten OS-specific branches. Use direct platform-specific APIs only behind narrow adapter boundaries and only when a cross-platform abstraction is insufficient. A dependency that deletes platform-specific code and test matrix burden is a simplification, not bloat.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haowjy/meridian-cli](https://github.com/haowjy/meridian-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
