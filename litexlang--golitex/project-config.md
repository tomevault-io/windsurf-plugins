---
trigger: always_on
description: Always apply these rules when working in this repository.
---

# Litex Local Instructions

Always apply these rules when working in this repository.

## Project Direction Through September

The main project direction through September is to use real mathematical
translation work as a pressure test for Litex. The target sources include
Mathematics in Lean, Terry Tao's Analysis I, miniF2F, MATH500, high-school
mathematics datasets, and Weil's Number Theory for Beginners.

The purpose is twofold:

1. Build strong evidence that Litex can express and verify meaningful
   mathematics quickly.

2. Use translation failures to discover real language, standard library,
   kernel, inference, automation, and diagnostic gaps.

Treat this as a structured feedback loop, not as a line-by-line porting
project. For each source, start with a small vertical slice before attempting
large-scale coverage. A useful slice is around 20-50 representative problems,
definitions, or theorem statements.

For each translated item, follow this loop:

1. Understand the mathematics first.

2. Write a natural Litex statement and proof attempt that matches the current
   verifier style.

3. Run the verifier and read the exact output.

4. Make the next smallest correction.

5. Classify the result as one of:
   - `translated`: the mathematical statement is naturally expressed in Litex.
   - `checkable`: the statement and proof are fully verified by Litex.
   - `blocked`: the failure reason is understood and recorded with a minimal
     reproduction.

Classify blockers explicitly. Useful blocker labels include:

1. `blocked_by_language`: Litex cannot naturally express the object,
   binding structure, or proposition yet.

2. `blocked_by_stdlib`: the proof needs missing definitions, lemmas, or
   theorem organization.

3. `blocked_by_infer_rule`: the mathematical step is simple but needs a new
   infer rule or builtin rule.

4. `blocked_by_kernel`: the verifier, runtime, well-definedness logic, or
   core proof model is missing required behavior.

5. `blocked_by_syntax`: the parser or syntax makes the intended expression
   awkward or impossible.

6. `blocked_by_diagnostics`: the verifier output is too indirect, confusing,
   or misleading to support a tight feedback loop.

7. `blocked_by_formulation`: the source statement needs a more natural Litex
   formulation rather than a mechanical translation.

Prefer early work on low-dependency, high-feedback corpora such as MATH500,
high-school math, and small miniF2F slices. Use Mathematics in Lean as a
standard library roadmap. Use Tao's Analysis I and Weil's Number Theory for
Beginners as deeper stress tests for structured definitions, chapter
dependencies, and long-form mathematical development.

Successful translations should become examples, benchmarks, or documentation
snippets when appropriate. Failed translations should become minimal blockers
that guide standard library work, language design, kernel improvements, or
better diagnostics. It is acceptable to use `know` or `abstract_prop` only when
the blocked part is clearly labeled and the rest of the development remains
explicit and checkable.

By September, a good outcome is not only a large number of translated items. A
good outcome is a working translation pipeline, checkable examples across the
main source families, a clear standard library gap map, a benchmark set for
Litex's mathematical ability, and minimal reproductions for the important
blockers.

For every source folder under `scripts/` or a similar local translation
workspace, maintain a nearby `todo.md` as the local blocker list for that
source. When translation work reveals a missing definition, theorem, infer
rule, builtin rule, syntax feature, or diagnostic gap, append a concise item to
that folder's `todo.md`. When one of those items is implemented or no longer
blocks the work, remove it from that `todo.md`. For example, if work in
`scripts/minif2f_tmp/` hits a missing feature, record it in
`scripts/minif2f_tmp/todo.md`; if the feature is later added, delete the
completed item.

## Dataset And Textbook Problem-Solving Loop

Use this loop for MiniF2F, MATH500, high-school datasets, Mathematics in
Lean, Tao Analysis, Weil Number Theory, and any other dataset or textbook
translation work. The goal is a pressure-test workflow, not only a final
answer.

For each item, proceed in this order:

1. First explain the natural-language mathematical idea. Work out the key
   transformations, cases, witnesses, estimates, or theorem dependencies before
   writing Litex.

2. Translate that mathematical plan into a natural Litex formulation. Prefer a
   proof shape that the current verifier can check: explicit equality chains,
   small intermediate facts, finite case splits, witnesses, named theorem
   calls, and local reusable lemmas.

3. If the proof cannot be completed immediately, write the best partial Litex
   proof first. It is acceptable to use `know` temporarily, but only for the
   blocked step. Next to each temporary `know`, add a concise comment saying
   why the step is not yet proved and what kind of missing support it appears
   to need.

4. Put unfinished attempts in the local unfinished-explanation area. In
   MiniF2F this is
   `scripts/litex-minif2f/unfinished_problems_and_why_they_are_unfiniishded/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litexlang/golitex](https://github.com/litexlang/golitex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
