---
trigger: always_on
description: - **Under construction** — Rust tooling trusted; Ori tooling (lexer, parser, typeck, eval, test runner) is NOT. Bugs are in the compiler, not user code.
---

- **Under construction** — Rust tooling trusted; Ori tooling (lexer, parser, typeck, eval, test runner) is NOT. Bugs are in the compiler, not user code.

## The One Rule: Correctness Above All

**Every decision you make must optimize for correctness.** The most correct, clean, and proper fix is the ONLY acceptable fix. There are no countervailing concerns:

- **Effort** is irrelevant. If the correct fix touches 10 files across 3 crates, that IS the fix.
- **Time** is irrelevant. A proper fix that takes longer is the only option.
- **Cost** is irrelevant. Complexity in service of correctness is not over-engineering.
- **Scope** is irrelevant. There is no "out of scope" — there is only correct and incorrect.
- **Risk** is irrelevant. "Safer to do the simple thing" is choosing incorrectness.
- **Responsibility** is irrelevant. There is no "not my code" or "different subsystem."
- **Current task** is irrelevant. "I was working on something else" is not a reason to do less.
- **Ownership** is irrelevant. There is no ownership — every file, every crate, every subsystem is yours.
- **Relatedness** is irrelevant. "That's a separate issue" does not exist.

When you see two possible fixes — one simpler and one more correct — the simpler one does not exist. You do not weigh tradeoffs between correctness and convenience. There is no tradeoff. Correctness wins. Always.

**Proper fixes only** — no workarounds, hacks, shortcuts, or temporary fixes. If the correct fix requires architectural change, that IS the work, not a reason to defer. If it requires refactoring a subsystem, that IS the fix, not a future improvement.

## Ownership & Deferral

- **One system, one owner: YOU** — compiler, typeck, eval, codegen, tests, spec, stdlib are one machine. Every bug, every failure, every interference is YOUR problem. You don't get to say "that's a different bug" or "that was already broken" or "my fix didn't cause that." If you found it, touched it, or your fix surfaced it — you own it. Fix every issue encountered. Add discovered issues to todo list.
- **ZERO DEFERRAL on bugs** — when you discover a bug, issue, leak, performance problem, or anything broken: every bug MUST get a concrete tracked artifact immediately. No mental notes, no "I'll remember," no comments-only. The two valid responses are:
  - **Blocking or critical/high:** fix it NOW using `/fix-bug` (creates a fix section file with plan-section rigor: root cause analysis, TDD matrix, completion checklist including TPR + hygiene review). The discovery IS the assignment.
  - **Non-blocking medium/low or unrelated to current task:** file it NOW using `/add-bug` (creates a tracked `- [ ]` entry in the bug tracker with repro, subsystem, severity). Filing via `/add-bug` is NOT deferral — it creates a concrete artifact that `/review-bugs` will triage. Deferral is when a bug has no artifact at all.
  No "tracked for later" (without an artifact), no "known issue" (without filing), no "pre-existing" (as justification for skipping). **Pre-existing bugs MUST be tracked immediately** — "pre-existing" is diagnosis only, never justification for ignoring.
- **Comments are NOT documentation** — a code comment (`//`, `#[ignore = "..."]`, TODO) is non-visible and non-actionable. It does NOT count as documenting an issue. Discovered bugs that cannot be fixed immediately MUST be added to the active plan or roadmap as `- [ ]` checkbox items. A comment alone is NEVER sufficient — comments are invisible to the planning system.
- **Tests that expose bugs = bugs found** — when writing tests (especially matrix tests), a failing test IS a bug discovery. Do NOT "fix the test to work around the bug" or say "this is a separate bug" and continue writing more tests. The moment a test reveals a bug: (1) STOP writing more tests, (2) file the bug via `/add-bug` (creates a tracked entry in the bug tracker) or add a `- [ ]` item to the active plan if the bug is in-scope, (3) THEN decide: if the bug blocks the current task or is critical/high severity, fix it now via `/fix-bug`; otherwise continue testing with the bug tracked. Rewriting a test to avoid a compiler bug without recording the bug is deferral — the test's purpose is to find bugs, and the bugs are the deliverable. "Completing the test matrix" is NOT more important than recording what the matrix found.
- **Proactive bug filing with `/add-bug`** — when you encounter ANY bug not related to your current task, invoke `/add-bug` immediately. Do NOT gloss over it as "not related", note it mentally and move on, or say "separate issue" without filing. If in doubt, file it — verification happens at `/review-bugs` time. A false positive costs nothing; a missed bug costs everything. Triggers: unrelated test failures, suspicious behavior, spec/impl mismatches, wrong error messages, fixable `#skip` reasons, TODO/FIXME describing unfixed bugs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/upstat-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
