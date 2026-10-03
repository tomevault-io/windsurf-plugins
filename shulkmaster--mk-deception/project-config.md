---
trigger: always_on
description: This file is the operational entry point for coding agents working in this
---

# MK Deception agent guide

This file is the operational entry point for coding agents working in this
repository. The supported target is the USA GameCube release, `GQNE5D`.

## Repository rules

- Preserve unrelated worktree changes. Inspect `git status --short` before and
  after editing.
- Treat retail assembly, call sites, symbols, relocations, and object layout as
  evidence. Decompiler output is a hypothesis, not ground truth.
- Edit source under `src/`, declarations under `include/`, and project metadata
  only when the evidence requires it. Do not hand-edit generated files in
  `build/`.
- Keep matching source readable and structurally honest. Do not force registers
  with `register`, fake `volatile`, dead sinks, incorrect prototypes, invented
  fields, embedded assembly, or unstructured `goto`.
- Exception: a local `goto` is allowed as a last resort when all of these hold:
  - Structured alternatives (early return, `break`, flag, shared exit, helper
    inline) were measured and regress the match.
  - The shape has backing evidence, such as matched references of the same
    vendor code in other decomps (MSL `__dec2num` uses `goto done` in both
    bfbb and TP/dusk), or the `goto` gives simpler, more honest control flow
    than a contrived `do { ... } while (0)`, one-trip loop, or dummy flag that
    exists only to force a match.
  - The jump stays inside one function and targets a label in that same
    function. Prefer a forward jump to a shared exit or cleanup. Never jump into a nested
    block past initializations, never emulate a loop that `for`/`while`
    expresses, and never use `setjmp`/`longjmp` or computed gotos.

  Record the evidence (measured alternatives and the reference) in the task
  report, not in a function comment.
- Exception: adding a function to the assembly-sequence mechanism is an
  extraordinarily rare action and requires explicit user permission for that
  specific function. Proof that a function is genuine handwritten assembly is
  necessary but does not itself grant permission. With approval, the function
  may invoke a `SEQ_<function>()` macro generated under `build/` from that
  version's retail-derived assembly and may be added to
  `config/<version>/asm_sequences.json`. Do not commit instruction payloads,
  synthesize a fallback, or use this path for ordinary compiler-generated
  functions. Automated, unattended, or goal-driven matching work must skip a
  function once evidence shows that it requires assembly; it must not add an
  assembly sequence or seek to satisfy the goal through one without explicit
  user permission.
- Make one coherent matching change at a time, rebuild, and inspect the same
  objdiff mismatch before trying another change.
- Preserve or explicitly account for the final retail SHA-1 check. A fuzzy
  percentage alone is not validation.

## Post-attempt status policy

After every matching attempt (including a reverted trial or no-edit stop),
if the function remains below 100%, update one source comment immediately above
the affected function:

```c
/* TODO: [near miss] 98.84%; equivalent latch CFG remains; stop at coloring. */
```

Required format: `TODO: [status] quick explanation`. Canonical statuses:

- `borked`: algorithm, CFG, ABI, or layout is demonstrably wrong.
- `breakthrough needed`: unresolved structural cause; name missing evidence.
- `breakthrough`: structural cause fixed; name remaining mismatch/next check.
- `near miss`: behavior/structure agree; localized codegen/relocation residue.
- `blocked`: tool, input, or authorization prevents verification.

Describe retained source, not the rejected candidate. Include current objdiff
score when available and concrete residual/next action; use one or two lines.
Replace previous status instead of appending history. For shared edits, update
functions whose result/classification changes. Never infer an exact match from
fuzzy improvement. Comments do not replace whole-TU checks, full build, or retail
SHA-1.

Once a function reaches a measured 100%, remove all matching-progress TODOs and
comments associated with it, including old percentages, near-match notes, soft
ceilings, attempt history, and `TODO: [matched]` markers. Do not replace them with
a new 100% comment. Preserve comments that explain behavior, algorithms, ABI,
layout, or other code semantics; if a comment mixes explanation with progress
tracking, remove only the tracking content. Unrelated functional TODOs remain.
Keep verification evidence and the distinction between report-exact, data-value
exact, and link-exact in reports and project metadata, not function comments.

## Local agent workspace

Use the Git-ignored `.agent-work/` directory for local agent artifacts:

- `issues/<task>/`: issue notes, investigation logs, and reproduction details.
- `patches/<task>/`: proposed `.patch` or `.diff` files and application notes.
- `decomp/<campaign>/`: matching reports, audits, scores, and evidence indexes.

Create these directories on demand. Use descriptive lowercase task names with
hyphens; include the base commit, affected symbols/files, commands, and validation
results in each task's notes. Keep patch files inert until deliberately applied.
Do not put credentials, retail images, or tool checkouts here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShulkMaster/mk-deception](https://github.com/ShulkMaster/mk-deception) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
