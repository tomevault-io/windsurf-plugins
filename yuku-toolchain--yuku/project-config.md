---
trigger: always_on
description: This document defines the coding style and principles that every contribution to this codebase must follow. It is written for human and AI agents alike. Read it once, then keep it close. The rules below are not suggestions.
---

# AGENTS.md

This document defines the coding style and principles that every contribution to this codebase must follow. It is written for human and AI agents alike. Read it once, then keep it close. The rules below are not suggestions.

## Design Goals

Three goals guide every decision, in order of priority:

1. **Safety**
2. **Performance**
3. **Developer experience**

All three matter. Good style advances all three. Style is more than readability. Readability is table stakes, a means rather than an end. Where understanding is missing, style fills the gap.

## Simplicity and Elegance

Simplicity is not a concession to the other goals. It is the "super idea" that solves multiple constraints simultaneously to achieve elegance.

Simplicity is not the first attempt but the hardest revision. Spend mental energy upfront, proactively. An hour of design saves weeks in production.

> "Simplicity and elegance are unpopular because they require hard work and discipline to achieve." Edsger Dijkstra

## Zero Technical Debt

Do it right the first time. The second time may never come, and steady incremental progress depends on knowing that what shipped is solid.

Do not allow potential latency spikes, exponential-complexity algorithms, or other showstoppers to slip through. When a problem is discovered, solve it. Do not defer.

## Safety

### Control Flow

- Use only very simple, explicit control flow. Avoid recursion where iteration suffices; when recursion is unavoidable, give it a bounded depth and assert that bound. Bounded execution must be guaranteed.
- Use only a minimum of excellent abstractions, and only when they make the best sense of the domain. Abstractions are never zero cost, and every abstraction introduces the risk of leaking.
- **Put a limit on everything.** All loops and all queues must have a fixed upper bound to prevent infinite loops or tail-latency spikes. Where a loop cannot terminate (e.g. an event loop), assert this.
- Use explicitly-sized integer types like `u32`. Avoid architecture-specific types like `usize`. The one accepted exception is the seam with the Zig standard library: `std.ArrayList.len`, slice indices into `[]const u8`, and similar interop are typed `usize` by the language. Keep `u32` everywhere we own the type, and limit `usize` to those boundaries (no `@intCast` chains that just propagate the boundary outward).

### Assertions

Assertions detect programmer errors. Unlike operating errors, which are expected and must be handled, assertion failures are unexpected, and crashing is the only correct response. Assertions downgrade catastrophic correctness bugs into liveness bugs. They are a force multiplier for fuzzing.

- **Assert all function arguments, return values, pre/postconditions, and invariants.** A function must not operate blindly on data it has not checked. Assertion density must average at least **two assertions per function**.
- **Pair assertions.** For every property to enforce, find at least two different code paths where an assertion can be added. For example, assert the validity of a node's shape immediately before serializing it, *and* immediately after deserializing it back.
- A blatantly true assertion can be used in place of a comment when the condition is critical and surprising. It is stronger documentation.
- **Split compound assertions:** prefer `assert(a); assert(b);` over `assert(a and b);`. The former reads simpler and gives more precise failure information.
- Use single-line `if` to assert an implication: `if (a) assert(b);`.
- **Assert relationships between compile-time constants.** Compile-time assertions check design integrity *before* the program executes. They are extremely powerful.
- **The golden rule:** assert the **positive space** (what you expect) AND the **negative space** (what you do not expect). The boundary between valid and invalid is where interesting bugs live. Tests must cover both spaces exhaustively, including the transition from valid to invalid.
- Assertions are a safety net, not a substitute for understanding. A fuzzer proves only the presence of bugs, never their absence. Therefore:
  - Build a precise mental model of the code first.
  - Encode that understanding as assertions.
  - Write the code and comments to explain and justify the model to your reviewer.
  - Use fuzzing as the *final* line of defense.

### Memory

- **Allocate with intent, not by reflex.** Prefer arena allocators with a well-defined lifetime over per-object alloc/free. An arena makes the allocation pattern visible at the call site, eliminates use-after-free, and frees in one step.
- **Size buffers ahead of time.** When the upper bound of a collection is known or can be estimated, reserve capacity once instead of growing repeatedly. This keeps the data plane on a predictable hot path.
- **Reuse, don't reallocate.** Scratch buffers used during a single pass should be reset between iterations, not freed and re-grown. Hold them on a parent struct so reuse is the default.
- Long-lived caches and pools must have an explicit upper bound. Unbounded growth is a latency bug waiting to happen.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuku-toolchain/yuku](https://github.com/yuku-toolchain/yuku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
