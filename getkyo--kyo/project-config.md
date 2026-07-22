---
trigger: always_on
description: This file helps AI coding agents work effectively on the Kyo codebase. Read [CONTRIBUTING.md](CONTRIBUTING.md) first: it is the comprehensive reference for all conventions, patterns, and design decisions.
---

# Agent Guide

This file helps AI coding agents work effectively on the Kyo codebase. Read [CONTRIBUTING.md](CONTRIBUTING.md) first: it is the comprehensive reference for all conventions, patterns, and design decisions.

## Operating Premise (read before anything else)

**Only complete and correct solutions are acceptable.**

This is non-negotiable. It applies to every change at every scope ; a one-line fix, a refactor across a module, a campaign that touches three modules. The framework is the same:

1. **Identify the correct fix.** Diagnose root cause. The correct fix is the one a reviewer with full context would call right. It is not the cheapest fix; it is not the fix that fits a time budget; it is not the smaller-scope alternative offered as a "pragmatic choice."

2. **Scope the correct fix honestly.** Read every layer it touches. If a type-system change at one factory cascades into another module's engine, that is part of the scope. Do not estimate the visible surface; estimate the actual surface. If the actual surface is larger than first thought, say so.

3. **Execute the correct fix.** Across every layer, end to end. Tests pass. Docs reflect the new shape. No dead branches. No "we will polish this later."

4. **Never propose alternatives that trade correctness for scope.** If you find a smaller fix that achieves a similar user-visible outcome via a less-rigorous path, that is information for the conversation. It is NOT an option for you to silently take. Surface it; let the user pick. Default to the correct one.

5. **Never normalize incomplete results.** If a refactor is half-done because the remaining layers are large, the work is half-done. Do not commit it. Do not declare it shipped. Do not write a celebratory summary listing what was done while omitting what was skipped.

6. **A change that breaks tests, breaks compile, or breaks documentation is not a change. It is a partial change.** Roll forward to completion or revert and reset.

7. **When in doubt, ask.** Asking for the correct scope is cheaper than shipping the wrong one.

The phrase **"shall I take the smaller fix?"** is structurally an offer to be incomplete. Do not offer it. If only the larger fix is correct, do the larger fix. If a smaller fix IS the correct fix, it would not have surfaced as "smaller" ; it would have surfaced as the natural answer.

The phrase **"this is good enough for now"** has no place in a kyo change. Either it is correct and done, or it is not done.

### The "pending issue" / "known issue" stop-pattern is banned

If a campaign goal is "X works end-to-end" and validation surfaces multiple bugs, fixing 4 of 5 and writing the 5th up as a "known issue requiring deeper investigation" does NOT meet the goal. The campaign is half-done. The 5th bug IS the work.

Forbidden phrasings, structurally — when you catch yourself drafting one, treat it as a flag that you are about to commit the anti-pattern:

- "Known issue: …" / "Documented as a known limitation."
- "This requires sustained X-internals investigation beyond a validation session."
- "Fix path: either … or … . That work is substantially larger than this session."
- "We've surfaced enough; let's pause and assess."
- "Documenting in FINDINGS for follow-up."
- "Out of scope for this session."

Re-scope instead: the unfixed bug is the next concrete task. Pick the fix path, execute it. Validation isn't done when the diagnostic writeup is comprehensive; it is done when the demonstrable end-state is true (host connects, tool calls succeed, response shape matches the spec, …).

The only legitimate stops within an in-flight campaign are:

1. The user explicitly says stop / pivot / park.
2. The next step requires a destructive or scope-affecting action that policy requires confirmation for (push, force-reset, schema-breaking change with downstream impact).
3. A genuine block: 3 attempts have failed AND each failure mode was novel AND there is no information you could gather independently to make a 4th attempt better.

"This is hard," "this touches a layer I haven't worked in yet," and "this needs deeper investigation" are not blocks. They are work.

## Core Rules

### Working Mindset

How to work on Kyo, not just what the code looks like. CONTRIBUTING.md carries the full conventions; these are the principles behind them.

- **Think in values, not actions (FP).** A Kyo computation is a value you compose, not an action you run. The effect row (`< Sync`, `< Async`, `< Abort[E]`) is part of the contract: never widen or hide it to shorten a signature. Default to immutability (`val`, `Span` over `Array`). Use mutable state only when it is genuinely necessary (a hot-path read buffer, a local accumulator); when you do, give it the smallest possible scope and keep it well isolated, never a shared `var` reaching across methods or fibers. For state that must cross fibers, use `Atomic*`. Keep side effects inside effects and pure positions pure. Make functions total: model absence and failure with `Maybe`, `Result`, and `Abort`, never `null`, exceptions-as-control-flow, or partial functions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getkyo/kyo](https://github.com/getkyo/kyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
