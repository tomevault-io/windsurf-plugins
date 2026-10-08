---
trigger: always_on
description: bearshape checks runtime array shapes, dtypes and tree constraints through
---

# bearshape agents

bearshape checks runtime array shapes, dtypes and tree constraints through
beartype. Scientific Python developers and library authors use its annotations
at API boundaries alongside native backend types. Keep the public identity
lowercase `bearshape`.

This is the shared Codex/Claude instruction file. `CLAUDE.md` points here.
Human setup and commands live in `CONTRIBUTING.md`; tool usage lives in
`tools/README.md`. Read `docs/maintainers/production-readiness.md` and the
current plan before changing a contract. Historical plans record their original
evidence; current owner decisions take precedence.

## Working rules

- State assumptions before changing behavior. Resolve routine implementation
  choices within the approved scope; ask when a missing decision changes the
  public contract or acceptance criteria.
- Maintenance is now surgical by default. The rename is complete. No speculative
  features, duplicate helpers, unused abstractions or broad cleanup unrelated to
  the requested work.
- Preserve public annotation syntax and useful errors. Use explicit names,
  simple control flow and the existing 2-space Python indentation.
- Keep runtime behavior and static consumer behavior aligned. Passing syntax
  checks, skipped tests and inferred `Any` do not prove support.
- Do not spawn other agents unless the user requests delegation.

## Current release contract

- GitHub ownership transferred on 2026-09-10: use `beartype/bearshape` and
  `https://beartype.github.io/bearshape/`. Preserve original authorship and
  historical plan evidence. PyPI ownership and trusted-publisher configuration
  are separate and remain unverified; future publishing must trust owner
  `beartype`, repository `bearshape`, workflow `pypi.yml`, environment `pypi`.

- bearshape is independently versioned; the candidate is `0.1.0rc0`.
- The current dependency and test target is beartype `0.23.0rc0`, with metadata
  `>=0.23.0rc0,<0.24`. Full native composite-union rollback is still incorrect.
  The owner chose full composition through supported upstream integration and
  permits a later candidate. Keep rc0 validation until a concrete replacement is
  implemented and tested; update metadata, lock, preflight and CI together.
- Do not replace beartype, monkeypatch upstream internals, infer union boundaries
  from bytecode, hide the defect with an xfail, or restrict composition to make
  CI pass. `tools/probe_union.py` reproduces the open release blocker.
- pyright, mypy, ty and pyrefly are supported through the consumer harness on
  Python 3.10–3.14. New checkers, including zuban, need an explicit plan.
- Native CuPy static typing is explicitly limited, as accepted by the owner.
  Its protocol fallback does not prove native ndarray method inference. Actual
  GPU runtime evidence is recorded separately in the handoff report.
- Release readiness remains withheld until the union integration and release
  prerequisites are satisfied. Merge approval does not authorize publication,
  deployment, access changes, ownership transfer or messages to maintainers.

## Product boundaries

- Root imports must not load NumPy, JAX, Torch, CuPy or optree. Backend behavior
  belongs in its explicit module; do not widen the root API for shorter examples.
- Preserve named, anonymous, fixed, arithmetic and broadcastable dimensions,
  `Scalar`, `Value(...)`, backend aliases, `Like[...]`, scalar-like types and
  `Tree[...]`. Dimensions are runtime constraints, not static shape proofs.
- NumPy owns the broadest dtype/scalar/Like surface. Other backends use their
  own native arrays and selected converters. Like validation does not replace
  the caller's argument or retry failed conversions through a different backend.
- Bare `@check` manages memo scope. Combine it with `@beartype`, or use
  `@check(conf=...)`, to perform type checking.
- Keep runtime tree structure syntax distinct from checker-supported containers.
  optree uses the default registry; there is no custom namespace API.
- `Value` expressions are trusted developer contracts, not a security sandbox.

Memo ownership, frame discovery, failed-alternative rollback, async contexts,
decorator metadata, dtype normalization and static aliases need focused positive
and negative regressions. Preserve lifetime/weak-reference evidence when changing
state ownership. Read the feature-to-test map in the handoff report.

## Major-work workflow

For features, contract changes, packaging, CI, release work and substantial
refactors: create a feature branch and matching worktree, commit an ExecPlan
following `PLANS.md`, and open a PR before implementation. Keep independent
changes in focused PRs and document any dependency on another PR. Use the
existing task goal when one exists; do not start duplicate goals.

Keep the plan current and validate each milestone. Present a concrete result
before requesting merge approval. Reuse explicit approval already given in the
conversation for that scope; do not ask again solely because work crossed a
milestone or worktree. After approval and passing checks, merge, verify main and
remove only clean merged branches/worktrees after preserving evidence.

For stacked PRs, merging a head into its actual base can mark the PR merged.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beartype/bearshape](https://github.com/beartype/bearshape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
