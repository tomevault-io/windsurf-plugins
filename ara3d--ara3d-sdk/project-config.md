---
trigger: always_on
description: This is the operating guide for AI agents (and humans) writing code in this repository.
---

# AGENTS.md — Working in the Ara3D SDK

This is the operating guide for AI agents (and humans) writing code in this repository.
Read it before making changes. It encodes how this codebase is built and maintained.

This project is maintained by **one person** with virtually no outside contributions.
Optimize for that reality: code that is **simple, correct, easy to refactor, and cheap to
understand six months later** beats code that is clever, abstract, or "enterprise-grade".

---

## 1. The two rules that override everything

1. **The code must work.** Correctness first. A working simple version beats a broken elegant one.
2. **The code must be easy to improve and refactor.** Everything below serves this.

If a guideline here ever conflicts with these two rules, the two rules win.

---

## 2. The standard workflow (do things in steps)

Work in small, verifiable increments. Never combine "make it work" with "make it pretty"
in the same step.

1. **Add code + tests** for the smallest useful slice of behavior.
2. **Verify it works** — build and run the tests (see §8). Do not proceed on a red build.
3. **Plan the refactor** — note what should improve. Add `// TODO:` markers in code and a
   matching entry in [`docs/TECHNICAL_DEBT.md`](docs/TECHNICAL_DEBT.md).
4. **Save the state** — this is a natural stopping/commit point. The working version is preserved.
5. **Apply the refactor** — change structure, not behavior. Tests from step 1 prove you
   didn't break anything.

> **Never refactor on a red build.** If tests are failing, get them green first, then refactor.

When given a multi-step task, use a todo list and keep it current.

---

## 3. Core principles

First, the monorepo-wide **API-first design principles** (canonical list in the root
[`AGENTS.md`](../AGENTS.md)) apply to everything here:

- **Write code as if writing a public API** — these SDK libraries literally are one.
- **Eat your own dogfood:** consume existing SDK APIs before adding new ones; awkwardness in an existing API is a reason to improve it, not bypass it.
- **Design for relocation:** code should move cleanly between projects/layers — few, explicit dependencies.
- **Write for the next learner:** someone else must be able to learn and use it quickly.
- **Obvious usage:** correct use discoverable from signatures and names alone.
- **Types and affordances guide correct use:** illegal states unrepresentable; misuse a compile error where possible.
- **Path of least resistance = best practice:** the easiest way must be the right way.
- **Composition and reuse by default:** every new piece is a candidate building block.

These are ordered roughly by how often they apply.

- **Keep it simple at first.** Start with the most direct solution that could work.
- **Use as little code as possible** to achieve the goal. Less code = fewer bugs, easier refactors.
- **Make it work before you improve it.** Resist premature abstraction and premature optimization.
- **Avoid repetition.** The second time you copy-paste, stop and extract a helper.
- **Reuse code when it makes sense** — but do not contort code to force reuse.
- **Minimize side effects.** Prefer functions that take inputs and return outputs.
- **Minimize dependencies.** See §7. The core libraries are nearly dependency-free; keep them that way.
- **Identify and track areas for improvement** instead of fixing everything at once (see §6).
- **Minimize the chance of breaking things when adding code.** Add alongside; don't rewrite
  working code unless that is the task.

### Priority order for geometry code

For the geometry utility library (and modifier/geometry samples), evaluate every function
against these properties, **in this order** — earlier ones win when they conflict:

1. **Correct** — it computes the right answer.
2. **Composable** — it combines cleanly with other functions.
3. **Reusable** — it generalizes beyond the one call site.
4. **Functional** — inputs to outputs; prefer expressions.
5. **Side-effect free** — no mutation of inputs or shared state.
6. **Succinct** — as little code as the above allow.
7. **Easily verifiable** — obvious to read and test for correctness.

A more efficient or mutable variant is a **later** step, and should land as a *separate*
function that can be compared against the canonical functional implementation — never by
compromising the canonical one.

---

## 4. C# style for this repo

This codebase has a distinct, consistent style. Match it.

### Favor immutable extension functions on interfaces

The author's favorite code is **immutable extension functions on interfaces**. Prefer this
shape over methods buried in concrete classes. Example pattern from `Ara3D.Collections`:

```csharp
// Small interface, then behavior added via extension functions that return new values.
public static bool InRange<T>(this IReadOnlyList<T> self, int n)
    => n >= 0 && n < self.Count;

public static bool IsEmpty<T>(this IReadOnlyList<T> self)
    => self.Count == 0;
```

- **Interfaces should be as small as possible to still be useful.** Add capability through
  extension functions, not by widening the interface.
- Prefer **immutable** data: `readonly` fields, `readonly struct`, return new values instead
  of mutating.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ara3d/ara3d-sdk](https://github.com/ara3d/ara3d-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
