---
trigger: always_on
description: Manabrew is a Tauri desktop/web client for Magic: The Gathering, powered by a Rust rewrite of the [Forge](https://github.com/Card-Forge/forge) rules engine.
---

# Manabrew — Agent Guide

Manabrew is a Tauri desktop/web client for Magic: The Gathering, powered by a Rust rewrite of the [Forge](https://github.com/Card-Forge/forge) rules engine.

- **UI**: React + TypeScript under `src/`, Tauri shell under `src-tauri/`
- **Engine**: Rust workspace under `manabrew-rs/crates/`, the rules engine being ported from Java
- **Java reference**: `forge/` — read-only; the source of truth for parity
- **Parity harness**: `manabrew-rs/crates/parity/` — runs Rust and Java side-by-side and compares traces

The engine is incomplete. Most day-to-day work is **finding parity bugs** with `yarn parity` and fixing them.

## Prime directive: root-cause, not symptom

Every engine fix must restore long-term correctness of the underlying mechanic — not patch the one card that triggered the bug report. If a single card seems to need a special case, that is almost always wrong: the general rule lives somewhere in Forge — find it, port it, mirror it.

Before writing the fix:

1. Read the corresponding Java file in `forge/forge-game/`.
2. Identify the rule the Rust port is missing, not the symptom that exposed it.
3. Mirror Forge's logic — same file names, same symbol names, same control flow.

Symptom-only fixes will be rejected in review. Full workflow: `docs/agents/ENGINE_BUGFIX_WORKFLOW.md`.

## Code-quality discipline

The project is large; every line is a long-term liability. Before adding code:

- **Read first.** Inspect the Java counterpart and the existing Rust file. Use the `scan-feature-parity` skill to confirm names/paths.
- **Mirror Java structure exactly.** Same file names (snake_case), same module layout, same method names. Do not invent. See `docs/agents/PARITY_PHILOSOPHY.md`.
- **Extend before creating.** Prefer adding to the existing file that already mirrors the Java side over a new one.
- **No premature abstraction.** Three similar lines beat a clever generic. No defensive code at internal boundaries. No speculative error handling.
- **Bound the change.** A bugfix touching 12 files needs justification. If it can be one or two, do that.
- **Do not add comments.** This is a hard rule, not a preference — the maintainer has repeatedly rejected over-commented diffs. The default for any new code (fields, methods, constants, blocks) is **zero comments**. Do not write doc-comments for self-explanatory members. Do not narrate what the code does — the reader can read code. A comment is allowed _only_ when intent is genuinely unrecoverable from the code itself: a hidden invariant, a "keep in sync with X" parity constraint, a workaround for a specific upstream bug, a non-obvious quirk mirroring Java. Matching the comment density of surrounding code is **not** a justification. Never narrate the edit: no "this used to be X", no "previously we …", no "added to handle …". When unsure, write nothing. Good naming and small functions are the documentation.

## Navigation map — read this before every task

Sub-AGENTS.md files are not auto-discovered by Codex or by Claude Code's parent-directory scan. **Consult this table at the start of any task** and read every file whose scope your change touches.

| File                                                               | Read it before                                                                   |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| `src/AGENTS.md`                                                    | Any change under `src/`                                                          |
| `src/components/game/AGENTS.md`                                    | Any change under `src/components/game/` (game board, modals, panels, zones)      |
| `src/components/companion/AGENTS.md`                               | Any change under `src/components/companion/` (paper-play life tracker)           |
| `src-tauri/AGENTS.md`                                              | Any change under `src-tauri/`                                                    |
| `manabrew-rs/AGENTS.md`                                            | Any Rust engine work — workspace map and engine module map                       |
| `manabrew-rs/crates/manabrew-engine/src/ability/effects/AGENTS.md` | Adding or modifying a `*_effect.rs` (most parity work)                           |
| `manabrew-rs/crates/parity/AGENTS.md`                              | Investigating a parity divergence or editing `regression.json`                   |
| `forge/AGENTS.md`                                                  | Anything under `forge/` (read-only warning)                                      |
| `forge-harness/src/main/java/forge/harness/AGENTS.md`              | Any change under `forge-harness/` (parity/host/common package boundaries)        |
| `scripts/AGENTS.md`                                                | Adding or running a build/parity script                                          |
| `website/AGENTS.md`                                                | Any change under `website/` (landing at manabrew.app, docs at docs.manabrew.app) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [witchesofthehill/manabrew](https://github.com/witchesofthehill/manabrew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
