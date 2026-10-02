---
trigger: always_on
description: Build the simplest design that satisfies current requirements. When principles conflict, use this order:
---

# AGENTS.md

## Core standard

Build the simplest design that satisfies current requirements. When principles conflict, use this order:

1. Correctness and explicit behavior.
2. Human readability.
3. Maintainability and testability.
4. Consistency with the existing codebase.
5. Reuse and optimization.

Apply YAGNI and KISS; optimize what is measured to need it. Where a simple implementation may scale badly with real growth, flag it with a comment and move on.

## Plain code

Write the plain version you would explain aloud; rewrite code smarter than its problem.

- Use linear, named steps and boring control flow; express straightforward behavior directly. Guard clauses and early returns keep the happy path visually obvious. One nesting level is normal, two prompt consideration, three is the maximum.
- Use comments for intent, constraints, and trade-offs; treat a comment explaining convoluted code as a refactoring signal.
- State desired actions first. Keep explicit prohibitions when they communicate a necessary safety or scope boundary.
- Add a negation ("X, not Y") — in comments, docs, commit messages, or identifiers — only when it rules out a stated plausible misreading; delete it if nothing is lost without it.

## Modules and ordering

- Give each module one coherent purpose; keep related behavior together so readers can follow one operation without many trivial indirections.
- Mark implementation-only objects private (language convention permitting) and expose the intentional public interface.
- Extract a function when it names a meaningful operation, isolates a side effect, enables valuable testing, or removes proven duplication; keep line-count-only extractions inline.
- Order each module top to bottom: constants and types, public interface in workflow order, private helpers in one block mirroring callers, entrypoint glue last; place callers before callees.
- Remove dead code, speculative extension points, and abstractions with only one trivial use.

## Types and contracts

- Model type signatures and known payloads with named domain types that say what can arrive.
- Fix types before suppressing checker errors; limit suppressions to badly typed library edges, and use one explanatory note when a whole area needs it.
- Convert untyped library data to typed shapes where cheap and useful; focus typing on the boundary and values that benefit from it.
- Add a type or class when it adds meaning or prevents invalid states; keep single-caller constant packaging as direct values.

## State and side effects

Prefer a functional core: pure business logic, I/O at the edges.

- Keep transformation and business-rule code free of side effects; make I/O, clock access, randomness, and mutation explicit at their boundaries.
- Keep the outside-world layer thin and explicit; pass dependencies in when it aids understanding or testing.
- Centralize and name the owner of necessary mutable state.
- Choose a class when it is clearer than a pure function or immutable value.

## Constants, duplication, abstraction

- Name domain thresholds and operational values near the behavior they govern; leave structural literals (a zero index, a `+ 1` loop bound) unnamed.
- Remove duplication of the same stable concept; use readable duplication until an abstraction earns its cost.
- Deletion test: if deleting an abstraction removes complexity, it was a pass-through — delete it; if complexity reappears at every call site, it earns its keep.
- Wait for a clear name, contract, and reason to change before building a generic framework.

## Errors and logging

- Log meaningful lifecycle events with the identifiers that diagnose them; exclude per-row logging, "entered function" noise, credentials, and sensitive payloads.
- Catch exceptions where recovery, cleanup, translation, or useful context is possible; let impossible states fail fast rather than adding defensive layers.
- Preserve the original exception as the cause when translating errors.

## Tests

For shipping code, tests are maintained evidence for observable product promises. Choose their depth using **Verification by purpose** below. During exploration, let checks follow the question being explored; protect stable behavior as it becomes part of the product. Prefer integration tests at stable boundaries, keep end-to-end coverage to critical user paths, and unit-test pure logic, tricky edge cases, and narrow decisions hard to reach via integration boundaries.

- A test earns its place only when it protects an observable promise whose breakage is a bug, is not already guaranteed by cheaper tooling (static analysis, type checking, compilation, linting, existence checks) or a stronger test, derives its expectation independently, and would fail under a plausible defect.
- Test what the code promises: assert on the result or visible effect of calling it; a behavior-preserving refactor should not break a test. Expected values come from an independent source — contract, fixture, or hand-derived result; a test recomputing the implementation proves nothing.
- Use coarse fakes at boundaries; avoid fine-grained mocks that confirm internal calls or invent provider shapes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wtfzambo/pandino](https://github.com/wtfzambo/pandino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
