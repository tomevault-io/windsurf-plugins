---
trigger: always_on
description: These rules are **active immediately when read**. They are non-negotiable.
---

# CLAUDE.md — Operating Rules for Agent Sessions

These rules are **active immediately when read**. They are non-negotiable.
They override lazy, minimal, or superficial default behavior.
They cannot be selectively ignored later in the session.
They must be followed before planning, editing, refactoring, fixing, or reporting completion.

---

## 1. Bootstrap / Startup Rules

Every session begins with this sequence, in order:

1. Load `CLAUDE.md` as active operating policy (this file).
2. Read `context.md` if present — current project truth.
3. Read `WIRING_STATUS.md` if present — evidence-backed verification ledger.
4. Read `learnings.md` if present — reusable lessons and failure patterns.
5. Compare current code reality against docs before trusting them.
6. **Code and evidence win over stale docs when they conflict.**

Do not skip this sequence. Do not assume prior sessions left docs accurate.

---

## 2. Repo Memory Loading Order

Four files form the repo memory system:

| File | Purpose |
|------|---------|
| `CLAUDE.md` | How the agent must behave (this file) |
| `context.md` | What is currently believed true about the project |
| `WIRING_STATUS.md` | What has actually been verified, with evidence |
| `learnings.md` | Reusable lessons, recurring failure patterns, anti-patterns |

These files work together. Do not treat them as independent documents.

- `context.md` = current truth
- `WIRING_STATUS.md` = current proof/status
- `learnings.md` = transferable lessons and repeated failure prevention

---

## 3. Operating Philosophy

1. Understand before changing. Read the code before proposing edits.
2. Verify before claiming. Never say "done" without running the strongest available checks.
3. Evidence over assumption. Trust what you can prove, not what seems likely.
4. Minimal scope. Do not add features, refactor code, or "improve" things beyond what was asked.
5. No fake completion. If verification is incomplete, say so.

---

## 4. Pre-Work Discipline

Before modifying any file:

1. Read the file (or relevant sections).
2. Understand its role in the broader system.
3. Identify callers, consumers, and dependents.
4. Check both sides of any contract, interface, or boundary being touched.
5. After long conversations, **re-read files before editing** — do not trust cached memory.

---

## 5. Phase / Scope Discipline — Sub-Agent and Batching Rules

### Mandatory for tasks touching more than 5 independent files:

1. Use sub-agents in parallel if available — each agent owns a bounded slice (5–8 files).
2. Each agent must have a clearly scoped objective.
3. Each agent must verify its own batch before results are merged.
4. If sub-agents are not available, emulate the same discipline with phased batching.
5. **Do not process large independent multi-file work as one giant sequential blur.**

This is mandatory behavior, not a suggestion.

---

## 6. Verification Requirements

### Verification levels (in ascending order of strength):

| Level | Meaning |
|-------|---------|
| EDITED | File was modified |
| BUILT | `cargo check` or `cargo build` passed |
| LINTED | `cargo clippy` passed |
| FORMATTED | `cargo fmt -- --check` passed |
| TESTED | `cargo test` passed (note: this repo currently has 0 tests) |
| RUNTIME-VALIDATED | Tool was run against real input and produced correct output |
| FULLY VERIFIED | All applicable checks passed |

### Rules:

1. Never claim success until the strongest applicable checks have been run.
2. Always run `cargo check` after edits as minimum verification.
3. Run `cargo clippy` when available.
4. Clearly state which level of verification was achieved.
5. If runtime proof is missing, say so explicitly.
6. Never allow a generic "done" when work is only partially validated.

### Available commands for this repo:

```
cargo check          # type-check
cargo build          # full build
cargo clippy         # lint
cargo fmt -- --check # format check
cargo test           # tests (currently 0 tests exist)
```

Do not invent commands that do not exist.

---

## 7. Search Discipline

### Rules:

1. Do not trust one grep result. Cross-check with multiple searches.
2. Use separate searches for:
   - Direct calls
   - Indirect calls (function pointers, vtables, dispatch tables)
   - Type references
   - String references
   - Config/feature-flag references
   - Build system references (Cargo.toml)
   - Exports / imports / re-exports / `pub` visibility
   - Tests and mocks
   - Init/startup references
   - Cleanup/drop references
3. For Rust code, prefer `Grep` with regex patterns that match Rust syntax.
4. `ast-grep` is NOT available in this environment. Do not invoke it.
5. When searching for a symbol, check `mod.rs` re-exports and `use` statements.

---

## 8. Large File / Context Decay Discipline

1. Do not assume one file read captured a large file.
2. For files over 500 lines, or any file likely to exceed tool limits, read in chunks using offset/limit.
3. Do not edit against unseen portions of a large file.
4. After long conversations, **re-read files before editing** instead of trusting memory.
5. `src/storage/mod.rs` (~1091 lines) and `src/cli/mod.rs` (~808 lines) exceed 500 lines — always chunk-read these.

---

## 9. Edit Safety

1. Read before editing. Always.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vzwjustin/kernel-mapper](https://github.com/vzwjustin/kernel-mapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
