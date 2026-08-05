---
trigger: always_on
description: An extensible decoder-only LLM training framework. Sibling to
---

# KohakUwULLM

An extensible decoder-only LLM training framework. Sibling to
[KohakuLatentMaid](https://github.com/KohakuBlueleaf/KohakuLatentMaid) (diffusion)
and [KohakuTerrarium](https://github.com/KohakuBlueleaf/KohakuTerrarium) (agents);
same house conventions, same tech stack.

## Project Overview

The framework provides slots; you select or drop in what you need. Reproducing a
Llama-shaped dense model, a Gemma-shaped sliding-window model, or a DeepSeekMoE
sparse model is a **config**, not a code change. Adding a new norm / MLP /
attention / position encoding / router is a registry entry or a dotted import
path -- no core edits.

Targets: dense models up to ~1B, MoE up to ~3B, trained on 4x RTX 5090 (32 GB,
sm_120). Primary corpus is the KohakuVault caption/tag databases, rendered into
TIPO-style prompt-generation examples.

## Code Conventions

### File Organization
- Source: `src/kohakuwullm/`
- Training / bench / tokenizer scripts: `scripts/`
- KohakuEngine config files: `configs/`
- Docs: `docs/`
- Max lines per file: 600 (hard max 1000). Split modules before they grow.
- Highly modularized -- one responsibility per module.

### Coding style rules

**Code files carry only *what*. Every *why* and *how* belongs in `docs/`.**
Source is not a place to write documentation.

1. **Docstring — what a module / class / function is for.** A straightforward
   description, and the caller-facing contract (arguments, units, dtype/layout
   requirements, what is returned). Typically one to three lines.
2. **Comment — what a piece of code is doing.** Name the logical step, which
   matters most in a kernel, or say what the code cannot say for itself. One
   line; two at the outside.
3. **NO rationale, derivation, measurement, history, alternative-considered or
   memo/noting comments anywhere in source.** If it explains *why this way* or
   *how it works*, it goes to `docs/` and the code gets at most
   `See docs/<file>.md`.
4. NO editorial comments, and none addressed to a future reader as a note.
5. AVOID long multi-line inline comments. A comment longer than the code it sits
   above is wrong.

### Import Rules
1. No imports inside functions (except optional deps and lazy imports that avoid
   long init time -- e.g. Lightning in `training/__init__.py`).
2. Grouping order: built-in, third-party, `kohakuwullm.*`.
3. Within a group: `import` before `from`, shorter dotted paths first, then
   alphabetical.

### Python Style
- Target Python 3.10+. Modern type hints: `list`, `dict`, `tuple`, `X | None`.
  Never `List`, `Dict`, `Optional`, `Union` from `typing`.
- Prefer `match-case` over deeply nested `if-elif-else`.
- `black src/ scripts/` and `ruff check src/ scripts/` before commit.

### The two architectural rules

**Select, don't dispatch.** Configuration resolves a concrete class or callable
**once**, at build time, via `build(spec, REGISTRY)`. The result is held as a
plain attribute and called directly. There is no per-step `if mode == ...`
anywhere in a training loop. If you find yourself adding a runtime branch on a
config value, the branch belongs in `__init__`.

**The backbone is a pure function.** `LMBackbone` maps `(tokens, seq_info) ->
hidden`. It knows nothing about the loss. Objectives live in `LMHead` and the
trainer, so changing the objective never touches the trunk.

### Numerics rules (this repo trains in low precision)

- **Never trust a low-precision scalar reduction.** Summing 16k bf16 terms loses
  several percent. Reduce in fp32 -- `LMHead.token_loss` asks the fused CE for
  `reduction="none"` and reduces itself, precisely for this reason.
- **Every Triton kernel needs a precision check** against an fp64 reference, in
  *both* fp16 and bf16, forward and backward. See `docs/internals/kernel-dev.md`.
- **Judge error in ULP, not absolutely.** An absolute tolerance that passes in
  fp32 is meaningless in bf16. Use `bench.timing.ulp_error`, and pick its mode:
  `"elementwise"` for elementwise kernels, `"rms"` for GEMMs and reductions
  (where a near-zero output is cancellation, not a small true value).
- **Token counts are int64.** A run passes 2^31 tokens in under an hour and a
  float32 counter stops incrementing at 2^24.

### Post-impl tasks
1. Verify the rules above, especially in-function imports.
2. `black` and `ruff`.
3. Logically separated commits.

### Testing

**`tests/` is removed for now and will be added back deliberately.** Do not
recreate the directory, do not add test files, and do not treat a missing test
as a blocker. Docs that cite `tests/test_*.py` name a specification, not a path
(see `docs/README.md`).

Verify with a throwaway script under `/tmp` instead, and hold it to the same bar:
it must **fail on the unfixed code**. A check that passes before and after
proves nothing.

### Audit loop (multi-step work: REQUIRED)

Do not stop at "it runs". Loop until it converges:

1. **Implement** the slice.
2. **Construct the negative case** -- the bug you'd accidentally introduce --
   and confirm the current code is actually wrong on it.
3. **Run lint.**
4. **Audit the diff** for clear bugs (typos, off-by-ones), integrity bugs
   (invariants broken, state drifted), and behavior bugs (does what's typed but
   the wrong thing for the spec).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KohakuBlueleaf/KohakUwULLM](https://github.com/KohakuBlueleaf/KohakUwULLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
