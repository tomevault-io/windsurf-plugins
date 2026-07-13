---
trigger: always_on
description: Treat this file as an active operating contract, not as passive background
---

# Pop Lang Agent Instructions

## Operating model: keep the contract active

Treat this file as an active operating contract, not as passive background
context. Critical rules must remain in the agent's active working set throughout
the task, especially while making design decisions, writing tests, editing code,
and declaring completion.

Do not assume that a rule remains operational merely because it appeared earlier
in the context. Re-read and reactivate the relevant invariants at each decision
checkpoint. After long tool output, a context switch, or a substantial subtask,
restore the working set before continuing.

Do not compress this file into a lossy summary. The persistent working set below
is a navigation and reactivation layer; every detailed instruction later in this
file remains binding.

## Persistent working set

Keep all of these invariants active throughout every task:

1. **Architecture authorizes behavior.** The accepted architecture and latest
   accepted ADRs are the source of truth. Do not invent contracts or let existing
   code redefine them.
2. **Architecture precedes tests; tests precede implementation.** Close design
   gaps first, encode the accepted behavior in deterministic tests second, and
   implement the smallest conforming change third.
3. **Pop Lang remains Pop Lang.** Preserve its native, strongly and statically
   typed, Luau-shaped identity. Prevent JavaScript/Rust/C#/D/C++ syntax drift and
   release-blocking Lua regressions.
4. **No operational dynamic escape hatch.** Never introduce `Any`, `Dynamic`,
   unchecked lookup/calls, string-based resolution, implicit globals, or dynamic
   fallback opcodes.
5. **Semantic concepts remain distinct.** Preserve the Item → Module → Bubble →
   Package → Workspace hierarchy and do not collapse records, classes, tables,
   namespaces, Modules, Bubbles, or Packages into one runtime mechanism.
6. **Backends share one semantic contract.** Keep HIR and MIR backend-neutral;
   MIR governs LLVM, the MIR interpreter, and future VM behavior.
7. **Compile time and reflection stay constrained.** Preserve deterministic,
   budgeted, capability-limited compile-time execution and the absence of
   unrestricted runtime reflection.
8. **Preserve work and verify honestly.** Keep user changes, make focused edits,
   run checks proportional to risk, and never claim a check passed unless it was
   actually run.

When two possible actions differ, prefer the one that preserves more of these
invariants simultaneously. When an action would violate one, stop and resolve the
architecture or test inconsistency instead of silently proceeding.

## Mandatory task loop

Use this loop for every change:

1. **Orient:** identify the requested outcome, affected ownership boundaries,
   public contracts, and likely architectural impact.
2. **Load authority:** read the required architecture documents, directly related
   documents, accepted ADRs, and closed decisions.
3. **Search broadly:** use `rg`/`rg --files` to find every affected term, example,
   decision, diagnostic, test, and cross-reference.
4. **State the authorized behavior internally:** distinguish accepted behavior,
   open questions, architecture gaps, and implementation details.
5. **Reactivate the persistent working set:** verify that the intended approach
   still preserves every applicable invariant above.
6. **Add tests before implementation:** make the pre-feature implementation fail
   for the intended missing behavior.
7. **Implement minimally:** make the smallest focused change that satisfies the
   accepted contract and tests.
8. **Re-scan and synchronize:** remove contradictory terminology and update all
   affected architecture, examples, decisions, diagnostics, and conformance
   material.
9. **Validate:** run the narrowest sufficient checks and the mandatory
   architecture-regression checks for the change.
10. **Report truthfully:** state what changed, what passed, what was not run, and
    any remaining architecture gap.

## Reactivation checkpoints

Pause and reload the relevant detailed sections of this file:

- after reading large files or long tool output;
- after switching between architecture, tests, and implementation;
- before changing any public language, library, runtime, artifact, diagnostic,
  tooling, or compatibility contract;
- before selecting syntax, naming, ownership, visibility, IR, runtime, GC,
  reflection, or library design;
- before modifying or accepting a test expectation;
- before declaring the task complete.

At each checkpoint, ask internally:

- What accepted architecture authorizes this exact decision?
- Which persistent invariants are active here?
- Am I accidentally treating implementation, convenience, or an open question as
  authority?
- What positive, negative, regression, consistency, and cross-backend evidence is
  required?
- What contradictory old model must be removed or synchronized?

## Stop conditions

Stop implementation and resolve the issue first when:

- no accepted architecture or ADR authorizes the proposed public behavior;
- an open design question would need to be answered silently;
- accepted architecture, tests, and implementation disagree;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poplanguage/pop](https://github.com/poplanguage/pop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
