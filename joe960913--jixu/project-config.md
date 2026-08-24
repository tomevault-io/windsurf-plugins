---
trigger: always_on
description: These instructions apply to every human or AI agent changing this repository.
---

# Jixu Repository Instructions

These instructions apply to every human or AI agent changing this repository.

## 1. Read order

Before proposing or making a change, read:

1. `SPEC.md` — normative product behavior and architecture;
2. this file — repository working rules;
3. the relevant implementation and tests; and
4. any directly relevant ADR or package documentation.

Do not infer architecture from filenames, examples, or provider SDKs when the
specification already defines it.

## 2. Sources of truth

- The ordered durable Event log is the sole authority for a Thread.
- State is derived from Events.
- A Checkpoint is a disposable performance cache.
- A Signal is transient and non-authoritative.
- Provider conversation state, UI state, traces, and in-memory objects are not
  runtime authority.
- `SPEC.md` is the authority for public semantics and package boundaries at the
  current commit. It is a living specification, not an immutable artifact.

Never introduce a second Thread state machine, a second Event history, or a
second component that can independently decide canonical Thread status.

## 3. Canonical concepts

Use the terminology in `SPEC.md` exactly:

- `Harness` owns exactly one immutable Agent and binds Stores and Drivers to the
  Kernel, but is not durable authority.
- `Agent` is immutable configuration and never hands work to another Agent.
- `Thread` is one durable, multi-turn history for that Agent.
- `Kernel` is I/O-free domain logic.
- `Event` is an immutable durable fact.
- `Signal` is a transient observation.
- `State` is the deterministic Event projection.
- `Reducer` is pure.
- `Effect` requests external work.
- `Driver` performs an Effect.
- `Tool` acts.
- `Skill` supplies instructional context.
- `Checkpoint` accelerates recovery but is not authority.
- `Fork` creates a new Thread.
- `Replay` performs no Effects.

Do not introduce `session`, `conversation`, `run`, `workflow`, `job`, or `task`
as a synonym or wrapper for Thread. Multi-Agent orchestration, handoff,
supervisors, swarms, Agent graphs, and Agent-as-Tool are outside the Jixu core
model. If an application needs a different Agent, it creates a different
Harness rather than adding Agent routing to one Harness.

If a proposed concept overlaps an existing term, stop and simplify instead of
adding another noun.

## 4. Architecture invariants

All implementation must preserve these invariants:

1. Externally observable work follows
   `Event -> Reducer -> Effect -> Driver -> Event`.
2. An external Effect is durably requested before dispatch.
3. Reducers do not perform I/O, read clocks, generate random IDs, or call SDKs.
4. Adapters depend on core ports; core never imports adapters.
5. Replay is read-only and never dispatches a live Driver.
6. Fork creates a new Thread with explicit parent lineage.
7. Unknown event types and schema versions fail closed.
8. Secrets never enter Events, Checkpoints, errors, or Signals.
9. Exactly-once behavior is never claimed without an enforceable idempotency
   contract.
10. Normal users do not write Reducers or manually construct Events.

## 5. Spec-driven workflow

`SPEC.md` records the best current understanding of Jixu. Development is
expected to expose incorrect assumptions, missing failure modes, or simpler
designs. Treat that evidence as input to the specification instead of forcing
the implementation to preserve a stale idea.

Spec-driven means the specification and implementation evolve together in a
controlled order. It does not mean the first specification is permanently
correct.

### When implementation evidence disagrees with the spec

1. Reproduce or otherwise verify the mismatch with code, tests, upstream
   behavior, or a concrete constraint.
2. Decide whether the mismatch is an implementation bug, an adapter detail, or
   a wrong/incomplete product assumption.
3. Record the evidence and the affected `JX-*` requirements.
4. Update `SPEC.md` to the smallest coherent design that explains the evidence.
5. Preserve stable requirement IDs where their meaning remains intact. Deprecate
   or replace an ID explicitly; never silently reuse it for different semantics.
6. Update acceptance criteria and compatibility or migration notes.
7. Then change the implementation and tests to match the revised spec.

Do not contort code around a disproven requirement. Do not silently make the
code authoritative either.

An evidence-backed spec correction that stays within the accepted goals,
non-goals, public promise, and task scope MAY be made in the same change as its
implementation. A change to project goals, non-goals, the public promise,
security guarantees, compatibility policy, or milestone scope requires explicit
maintainer direction before implementation.

Significant decisions SHOULD receive an ADR explaining the evidence and rejected
alternatives. Minor clarifications do not need an ADR. Avoid speculative spec
churn without implementation evidence or a concrete user requirement.

Before implementation, classify the change:

### Scoped UI change

A localized UI bug fix, interaction correction, copy/spacing/color polish, or
component-level presentation change does not require a `SPEC.md` update or new

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joe960913/Jixu](https://github.com/joe960913/Jixu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
