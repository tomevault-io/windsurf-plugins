---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Agent Guidelines

@prompts/repl-driven-development.md
@prompts/common-lisp-expert.md

These two prompts are copied verbatim from the `cl-mcp` repository, so a few of their
assumptions do not hold here:

- `repl-driven-development.md` describes the **cl-mcp MCP tools** (`repl-eval`,
  `lisp-edit-form`, `clgrep-search`, …). They only apply when the `cl-mcp` MCP server is
  connected to the session; otherwise fall back to the built-in Read/Edit/Grep/Bash tools.
- Both prompts assume **Rove** as the test framework and `mallet` as the linter. This project
  uses Rove too (`asdf:test-system`, `rove`) but has **no lint step** — see *Commands* below.

## Overview

Common Lisp implementation of Random Forest for multiclass classification and univariate
regression, plus **Global Refinement** and **Global Pruning** of a trained forest
(Ren, Cao, Wei, Sun, "Global Refinement of Random Forest", CVPR2015). Multivariate regression
is not implemented.

## Commands

Load (requires `cl-online-learning` and `cl-libsvm-format` from the same author, checked out
into quicklisp/roswell `local-projects`):

```lisp
(ql:quickload :cl-random-forest)   ; package nickname: CLRF
```

Run the test suite (rove-based):

```lisp
(asdf:test-system :cl-random-forest)
```

```sh
./t/run-test.ros                   # what CI runs; roswell with dynamic-space-size=2048
rove cl-random-forest-test.asd     # rove's own runner -- prints nothing under CCL, see below
```

`rove <system>.asd` mutes `*standard-output*` and routes rove's report through a
synonym-stream lookup that resolves differently on CCL, so under CCL it runs the suite but
prints nothing and the exit code is the only signal. CI uses `./t/run-test.ros` for that
reason; prefer it when you need to see results.

Tests are split into feature systems, each of which can be run on its own:

| System | Tests |
|---|---|
| `cl-random-forest-test/dataset` | dataset download and conversion (3) |
| `cl-random-forest-test/decision-tree` | decision tree accuracy (2) |
| `cl-random-forest-test/forest` | random forest accuracy (2) |
| `cl-random-forest-test/refinement` | global refinement accuracy, learner-type plumbing and convergence detection (9) |
| `cl-random-forest-test/parallel` | parallelized training accuracy (4, SBCL only) |
| `cl-random-forest-test/regression` | univariate regression behaviour (5) |
| `cl-random-forest-test/pruning` | global pruning behaviour (5) |
| `cl-random-forest-test/packed` | packed inference representation (11) |

`cl-random-forest-test` is the aggregate that runs all eight.
`cl-random-forest-test/fixture` holds the shared dataset loaders and helpers and has no tests.

Load the feature system first (`ql:quickload` or `asdf:load-system`), then:

```lisp
(rove:run :cl-random-forest-test/forest)                              ; one feature
(rove:run-test 'cl-random-forest-test/forest::a9a-forest-accuracy)    ; one test
```

There is no lint step. CI (`.github/workflows/ci.yml`) runs the matrix
{sbcl-bin, ccl-bin} × {ubuntu-latest, macOS-latest}, less ccl-bin on macOS, which the
workflow excludes because macOS CCL binaries are no longer distributed — three jobs, not
four. Roswell has no ccl-bin for **ARM64 Linux** either, so on an aarch64 development
machine the CCL half of the matrix cannot be reproduced locally at all and CI is the only
place it runs. Treat a green local suite as evidence about SBCL only: CCL does not check
type declarations, and its `integer-decode-float` normalises a denormal's significand where
SBCL leaves it alone — both have produced CI-only failures in this repository.

Test/example caveats:
- `cl-random-forest-test/regression`, `.../pruning`, and the seven synthetic tests in
  `.../refinement` (`refine-learner-default-path-unchanged`, three `refine-learner-of-type-*`
  and three `refine-learner-process-*`) need no network: they use
  `cl-random-forest-test/fixture`'s deterministic synthetic data, or build their own.
  Everything else downloads datasets.
- `cl-random-forest-test/packed` uses the fixture's synthetic data and needs no network.
- Of the regression and pruning suites' ten tests, seven are **property assertions**, not
  pinned accuracy numbers, and three deliberately pin bugs that are still open: `regression-refine-learner-default-gamma-diverges`
  (issue #16), `pruning-strands-leaves-without-sample-indices` (issue #14) and
  `pruning-does-not-update-forest-n-leaf` (issue #15). Each says so in a comment. When one of
  them starts **failing**, the underlying bug has been fixed and the test should be replaced by
  a positive assertion rather than "repaired".
- Datasets are loaded lazily and memoized in `cl-random-forest-test/fixture`, so any single
  test can be run on its own and will fetch only what it needs.
- The three `refine-learner-process-*` tests synthesise a refine dataset directly rather than
  building a forest, so they run in under a second. They also carry deliberate label noise:
  on separable data the best and last epoch coincide and the contract they check is not
  observable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masatoi/cl-random-forest](https://github.com/masatoi/cl-random-forest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
