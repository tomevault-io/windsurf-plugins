---
trigger: always_on
description: Package manager for Obsidian vault templates. TypeScript, Pastel + Ink TUI, spec-driven development. The engine is agent-agnostic — shard content determines which AI agents are supported (Claude Code, Codex, Gemini CLI).
---

# ShardMind

Package manager for Obsidian vault templates. TypeScript, Pastel + Ink TUI, spec-driven development. The engine is agent-agnostic — shard content determines which AI agents are supported (Claude Code, Codex, Gemini CLI).

## How This Repo Works

This project is **spec-driven**. The architecture and implementation are fully designed before code is written. Claude Code reads the specs and implements them.

### Source of Truth

| Document | What | When to Read |
|----------|------|-------------|
| `VISION.md` | Origin story, architectural bets, scope guardrails, non-goals. | Before proposing features or scope changes. |
| `ROADMAP.md` | v0.1 milestones linked to GitHub issues. Build order. | Before starting a new milestone. |
| **`docs/SHARD-LAYOUT.md`** | **v6 shard-layout contract + three binding invariants. Active design spec.** Folded into `ARCHITECTURE.md §3` and `IMPLEMENTATION.md §4.5` / `§4.5a` / `§4.5b` for the engine specs; this doc remains the canonical contract for the binding properties + author-facing layout. | Before implementing anything related to shard layout, install walk, hook context, adopt command, or obsidian-mind v6. Authoritative over ARCHITECTURE.md / IMPLEMENTATION.md where they conflict. |
| `docs/ARCHITECTURE.md` | The what and why. 22 sections. Core concepts, ownership model, schema format, module system, values layer, signals, operations, competitive moat. | Before making any architectural decision. |
| `docs/IMPLEMENTATION.md` | The how, exactly. System diagram, data flows, module specs with TypeScript signatures, algorithms as numbered steps, error cases, 20 merge test fixtures, 6-day build plan. **§9 (Build Plan) is stale — see [#70](https://github.com/breferrari/shardmind/issues/70) for the current task list.** | Before implementing any module. |
| `docs/COMPONENTS.md` | Iterated UI patterns (A/B), `useOncePerKey` hook, `rerender()` regression-test convention. Codifies why state-machine-iterated prompts (`AdoptDiffView`, `DiffView`) need per-iteration ref scoping vs. boolean refs. | Before adding a new Ink component that may be iterated by a parent state machine, or before changing a `useRef` shape inside one. |
| `examples/minimal-shard/` | Minimal test shard for development. 4 values, 2 modules, signals. Flat v6 layout (`.shardmind/` sidecar, content at native paths, dotfolder `.njk` for rendering). | Use as a fixture for engine-level tests; obsidian-mind v6 conversion lands at Milestone 5. |

**Read the relevant spec section before writing code.** The specs define inputs, outputs, algorithms, error cases, and test expectations for every module. Don't improvise — implement what the spec says.

### Build Order

**Authoritative task list for v0.1: [#70](https://github.com/breferrari/shardmind/issues/70)** (engine changes, shard conversion, docs, tests, acceptance criteria). Spec for what to build: [`docs/SHARD-LAYOUT.md`](docs/SHARD-LAYOUT.md). The day-by-day rhythm in `docs/IMPLEMENTATION.md §9` is preserved below for cadence reference, but the specific sub-tasks within each day are superseded by #70's tracks (the `templates/` walk, `partials` field, and Cookiecutter-style source/target split described in §9 do not reflect the v6 contract).

| Day | Focus |
|-----|-------|
| 1 | Scaffold + core modules per #70 "Walk + discovery" and "Schema + values" tracks |
| 2 | Install command with full Ink wizard + module review + `--defaults` flag |
| 3 | Merge engine — TDD with 17 fixtures (write fixtures FIRST, then implement) |
| 4 | Update command + status display + verbose mode + ref re-resolution + `--version` / `--include-prerelease` |
| 5 | obsidian-mind v6 conversion (`.shardmind/` sidecar, dotfolder `.njk`, hooks) + `shardmind adopt` command |
| 6 | Research-wiki shard, Invariant 1 E2E test, polish, npm publish |

## Working Agreement (v6 execution standard)

Every v6 sub-issue (#73–#78, #14, #15, #85) passes these gates before merge. These practices are what this project has used from day one — spec-driven, fixture-first, adversarial. This section makes them explicit so any session picking up work knows the bar.

### 1. Spec before code

- Read the relevant section of [`docs/SHARD-LAYOUT.md`](docs/SHARD-LAYOUT.md) AND the linked issue body before touching code.
- If the spec is silent or ambiguous on a decision you need, **update the spec first via a separate commit** — do not invent behavior in the implementation.
- If your implementation reveals a spec mistake, fix the spec and submit the fix alongside the code change.

### 2. Tests before implementation

- Write the failing test first for every new behavior or bug fix. No code without a test.
- For merge-engine-class work (`drift.ts`, `differ.ts`, `renderer.ts`), write **fixtures first** — the pattern used for the 20 merge fixtures in `tests/fixtures/merge/`.
- Prefer property-based tests via `fast-check` when the input space is wide: ref-syntax parsing, `.shardmindignore` glob matching, hash-equivalence under whitespace.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breferrari/shardmind](https://github.com/breferrari/shardmind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
