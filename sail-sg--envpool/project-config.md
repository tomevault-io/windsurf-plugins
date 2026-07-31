---
trigger: always_on
description: This file is for agents adding a new environment family or upstream task family
---

# EnvPool Agent Guide

This file is for agents adding a new environment family or upstream task family
to EnvPool.

## Goal

Ship a native EnvPool implementation that is:

- C++ at runtime
- fully registered
- documented
- deterministic
- pinned to a specific official oracle/version when one exists
- alignment-tested against the official oracle when one exists
- render-tested when rendering is supported
- green in full-platform CI and release packaging

Do not stop at a partial port.

## Hard Rules

- Do not add a Python runtime bridge for environment logic.
- Do not import the official Python environment inside C++ implementation code.
- If there is an upstream family with multiple registered IDs, support all of
  them before calling the port complete.
- Match the public EnvPool IDs to the upstream task family shape. If the
  upstream exposes multiple named scenarios / env IDs, register them
  individually instead of collapsing them into one generic task.
- Official Python packages may be used only in tests and doc-generation tooling
  as an oracle.
- Pick the exact upstream oracle/version before implementation and keep tests
  anchored to that target. Do not mix comparison baselines across upstream
  versions in one family.
- Keep changes localized to the new env family unless shared infrastructure is
  strictly required.
- Prefer following the closest already-green in-tree alignment/render harness
  before inventing a new comparison strategy. Reuse proven patterns such as the
  MuJoCo DMC-style oracle and render checks when the family shape matches.
- Prefer reusing existing `third_party/` libraries before adding new ones.
  Do not duplicate SDL-like dependencies just because a new env family needs
  them.
- Prefer fetching official upstream/library source in `third_party/` over
  copying source blobs into the repo. If code must be patched, store the patch
  under `third_party/<family>/`.
- Keep env-specific packaging declarative when possible.
  - prefer `setup.cfg`, Bazel data deps, and filegroups
  - avoid env-specific imperative hooks in `setup.py`
- Do not hide large upstream patch programs inside `envpool/workspace0.bzl`.
  `workspace0.bzl` may wire repository rules, but env-family patch payloads
  should live under `third_party/<family>/`.
- If an upstream registry list or scenario list is needed, generate it from the
  upstream source in tooling or `third_party/` rather than maintaining a copied
  handwritten list in runtime code.
- When extending an existing family with new upstream tasks, treat the new work
  as incomplete until the whole intended upstream task set is wired through the
  existing family harness. Do not stop at one representative env because it was
  enough to prove the plumbing.
- When extending an existing family, follow the already-green setup used by the
  older tasks in that family.
  - keep the same oracle/reset-sync strategy unless there is a documented
    reason not to
  - keep the same deterministic/render validation shape unless the new task
    fundamentally cannot support it
  - keep the same GL/bootstrap strategy before inventing a platform-specific
    workaround
- If a tolerance is unavoidable, make it:
  - as small as possible
  - platform-scoped if possible
  - documented in the test
- Do not widen tolerances or add platform-only workarounds until the root cause
  is understood well enough to explain why bitwise matching is impossible.
  "macOS only" is not an acceptable first fix.
- Do not use `skipTest` to hide a new-env regression on a supported platform
  when the family already has a known-good render / GL setup. Exhaust the
  existing in-tree strategy first, then document the real blocker if one
  remains.
- Before adding a broader tolerance for a new task family, inspect nearby
  existing envs in the same suite to confirm the issue is truly isolated. A new
  task sitting at `1e-5` while older tasks hold `1e-11` is a root-cause
  investigation signal, not a reason to immediately relax the whole suite.
- Minimize packaged assets for upstream-backed families. If only a subset of
  upstream assets is actually needed at runtime, exclude the rest and verify
  that release packaging still works.

## Implementation Loop

1. Read the existing human docs first:
   - `docs/content/new_env.rst`
   - `docs/content/contributing.rst`
2. Pick the closest in-tree reference implementation.
   - `envpool/dummy/` for skeletons
   - a real env family with similar action/obs/render structure
3. If the family wraps an upstream project, design the `third_party/<family>/`
   layout first.
   - repo rule / archive wiring
   - BUILD glue
   - patch files
   - generated upstream metadata helpers if needed
4. Create the env family folder under `envpool/<family>/`.
5. Add the minimum file set:
   - `BUILD`
   - `__init__.py`
   - `registration.py`
   - C++ headers / sources
   - family tests
6. Implement the native spec first.
   - config
   - observation/state spec
   - action spec
   - reward / termination / truncation
   - info keys
7. Implement native stepping logic in C++.
8. Implement native rendering in C++ if the family supports render.
9. Register every supported env ID and EnvPool alias.
10. Add docs:
   - env doc page under `docs/env/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sail-sg/envpool](https://github.com/sail-sg/envpool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
