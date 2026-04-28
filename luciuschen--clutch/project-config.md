---
trigger: always_on
description: Elisp best practices distilled from llm.el, magit, consult, eglot, vertico/marginalia.
---

# clutch Development Guide

Elisp best practices distilled from llm.el, magit, consult, eglot, vertico/marginalia.

## Core Principles

- **Question every abstraction**: Before adding a layer, file, or indirection, ask whether it solves a current problem. If the answer is hypothetical, do not add it.
- **Simplify relentlessly**: Three similar lines are better than a premature abstraction. A single large file is better than several tiny files with unclear boundaries.
- **Fewer files, clearer boundaries**: Split only when a file has a genuinely distinct responsibility. Never split for cosmetic reasons.
- **Delete, don't deprecate**: Remove unused code entirely. No backward-compatibility shims, re-exports, or "removed" comments.
- **Converge UX**: Prefer one clear entry point and one consistent behavior model over overlapping commands or branchy mode-specific behavior. Wrapper commands are fine, but they must share one resolution path, one action registry, and one default-action model.

## Diagnosis and Change Discipline

- **Find the root cause before changing behavior**: Do not patch UI timing, cache invalidation, or command flow until you can name the failing layer and explain why it is responsible.
- **One failed fix narrows the hypothesis**: If the first attempted fix does not hold, reduce the hypothesis space and gather evidence. Do not stack another speculative patch on top.
- **Two failed fixes stop the patching loop**: After two failed fixes on the same issue, stop changing behavior and switch to diagnosis only.
- **Fix the right layer**: If the real problem belongs in the JDBC agent, protocol code, cache model, or connection lifecycle, move the fix there instead of compensating in the UI layer.
- **Stabilize workflow changes before coding**: For any change that alters a primary entry point, default action, or action menu, write a short design note first. Keep object resolution, action definition, and action presentation separate.
- **Keep experiments narrow**: Start new directions with the smallest slice that proves the workflow is worth having. Do not expand scope before the first slice shows real user value.
- **Flag compensating code as design debt**: When touching a subsystem, look for code that compensates in the wrong layer — `condition-case nil` swallowing internal errors, re-querying data already available from a caller, timing hacks, or silent fallbacks. These are not blockers; record them in a postmortem as design debt rather than fixing inline. Do not let debt discovery delay the current change.

## Error Handling and Testing Discipline

- **Errors must surface, not hide**: Do not add fallback/default returns that silently swallow failures. Let errors propagate immediately.
- **Catch at the boundary, nowhere else**: Only the outermost API layer (process loop, top-level command handler) should catch and convert exceptions to error responses. Business logic must not `condition-case` around internal calls.
- **Tests must fail when the code is wrong**: If deleting or breaking the function under test does not turn the test red, the test is worthless. Assert specific, distinguishable output values.
- **No hard-coded expectations**: Use diverse inputs — multiple data sets, random values, boundary cases — so that a hard-coded return cannot satisfy all assertions.
- **Red before green**: When fixing a bug, first write a failing test that reproduces it. Confirm it fails. Then fix the code. A test written after the fix has never been proven to catch the bug.

## Architecture and Implementation

- **Interface / implementation separation**: `mysql.el` and `pg.el` are pure protocol libraries with no UI. `clutch.el` depends on `clutch-db.el`, not protocol layers directly.
- **Single responsibility per file**: Do not mix protocol code with rendering code.
- **Keep `clutch.el` as the entry point**: External consumers should continue to load `(require 'clutch)`. When implementation moves out, `clutch.el` becomes the assembler, not a grab bag.
- **Split by stable workflow boundaries**: Prefer modules such as result UI, object workflow, staged mutation flow, or schema/cache lifecycle. Do not split by vague internal labels like `common`, `utils`, or `helpers`.
- **Move whole responsibilities, not leftovers**: A file split must take a real slice with its state, commands, and render helpers together. If the original file still owns the behavior and the new file only adds glue, the split is not done yet.
- **Do not split for cosmetics**: A shorter `clutch.el` is not enough reason to extract a file. Split only when ownership becomes clearer and future changes will touch fewer files.
- **Stop splitting before glue takes over**: If a proposed extraction mostly adds `defvar`, `declare-function`, and cross-file hopping without reducing conceptual ownership, stop. That is a sign of over-modularization.
- **Use declarations to keep modules honest**: When a module depends on shared globals or functions defined elsewhere, add explicit `defvar` / `declare-function` forms so byte-compilation stays clean.
- **Favor incremental modularization**: Move the smallest coherent slice first, then reload, byte-compile, and rerun focused tests before attempting the next extraction.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuciusChen/clutch](https://github.com/LuciusChen/clutch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
