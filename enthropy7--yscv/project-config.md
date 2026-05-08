---
trigger: always_on
description: This file is the contract between this repository and any AI coding
---

# AGENTS.md — rules for AI coding agents working on yscv

This file is the contract between this repository and any AI coding
agent (Claude Code, Copilot Workspace, Cursor, Aider, etc.) that
proposes changes here. Concrete, enforceable, short.

The same rules apply to human contributors.

---

## Responsibility

Every PR has exactly one responsible party: the human who opens it.
This holds whether the diff was hand-typed, drafted by an agent, or
copy-pasted from a chat log. By submitting, you assert four things
about the change:

1. You have the right to submit it under this repository's licence.
2. You have read the code you are submitting and understand what it
   does. If asked "why this line?", you can answer without
   re-prompting an agent.
3. You have run the local gates listed in [section 2](#2-the-workflow)
   and the result is what you say it is. "The agent said tests pass"
   is not a substitute for running them.
4. For any performance claim, you measured it yourself on your own
   hardware and the numbers in the PR description are yours.

There is no "the agent did it" defence. A regression introduced by
an agent-drafted patch is the submitter's regression, and reverting
or fixing it is the submitter's job.

Disclosure of method: if a change is non-trivial and was drafted
with an agent, mention that in the PR description in one line —
what the agent did, what you verified. This is for context, not
credit; do not add `Co-Authored-By:` lines for the agent (see
[section 3](#3-the-do-not-list)).

Anonymous or pseudonymous PRs that exist only to relay agent output
without a human standing behind them will be closed unread.

---

## 0. Read these first, in order

1. [`CONTRIBUTING.md`](CONTRIBUTING.md) — project priorities and the
   change workflow.
2. [`docs/feature-flags.md`](docs/feature-flags.md) — what each
   Cargo feature does, when to enable it, runtime env knobs.
3. [`docs/architecture.md`](docs/architecture.md) — the high-level
   shape of the workspace, dispatch boundaries, dataflow.
4. The crate-local `README.md` of the crate you are touching.
5. The latest `docs/perf-arc-*.md` if you're touching a hot path —
   it lists what landed, what was tried and reverted, and why.

That's about thirty minutes of reading and prevents the most common
class of "agent re-proposes a previously-tried dead-end" PRs.

---

## 1. The five commandments

Non-negotiable. Violating any one is grounds for the PR to be
reverted on sight, regardless of benchmark numbers.

1. Blazing fast or out. Every change on a hot path must show a
   measured win on the shape range it targets. "It compiles" and
   "tests pass" are not enough for an inference loop. Use criterion
   benches or a representative end-to-end harness; never push a
   perf-claimed change without numbers in the commit message.

2. Multi-arch SIMD or scalar. No SIMD function lands as x86-only
   or aarch64-only. Either ship NEON + AVX/SSE + scalar fallback
   together, or stay scalar. Runtime feature detection via
   `std::arch::is_*_feature_detected!`. The dispatch pattern is
   consistent across the codebase — copy from
   `crates/yscv-kernels/src/ops/matmul.rs` if in doubt.

3. Minimal code. Smallest correct change. Iterators where the
   compiler vectorises them, no `unwrap`, no `#[allow(dead_code)]`,
   no half-finished implementations. New abstractions only when
   they earn their keep — three similar lines beat a premature
   trait family. If you find yourself adding a helper to make the
   diff "look cleaner", delete the helper.

4. Numerical correctness is non-negotiable. Default paths produce
   bitwise-identical or 1-ULP-close outputs against the scalar /
   reference path on supported shapes. Approximations (fp16
   storage, mixed-precision compute, quantization, fast-math
   reductions) ship behind explicit env knobs or Cargo features,
   never as default. A commit that "speeds things up" by silently
   relaxing precision is a regression — measure the output drift
   in the same suite that measures latency.

5. Document the change in the same commit. If you change a public
   API or runtime behaviour, the docs change with the code: the
   crate `README.md`, `docs/feature-flags.md` for new flags,
   `docs/ecosystem-capability-matrix.md` for new capabilities,
   `context.md` for surface counts. The
   `bash scripts/check-doc-counts.sh` gate enforces this for the
   counted surfaces (crate count, ONNX op count, tensor methods,
   imgproc fns, etc.).

---

## 2. The workflow

1. Confirm scope. Restate to yourself in one sentence what is
   changing and why. If you can't, you don't have scope yet — go
   back to the issue or prompt and clarify.

2. Read the prior art. `git log --oneline -- <path>` for the files
   you're about to touch, and `git log --grep="<topic>"` for any
   prior arc on the same problem. Reverted commits and their
   commit messages are the canonical record of what didn't work.

3. Implement minimally. See commandment #3.

4. Test. Unit + integration where shape varies. Numerical ops
   need reference-parity tests against documented formulas.
   Hot-path code needs a criterion bench.

5. Bench. For perf changes, measure on the actual workload,
   three runs minimum, report median + min. The repo's bench

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enthropy7/YSCV](https://github.com/enthropy7/YSCV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
