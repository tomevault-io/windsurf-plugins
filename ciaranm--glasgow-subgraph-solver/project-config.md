---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code
in this repository.

It is deliberately short. Almost everything a developer needs here is project
documentation rather than agent guidance, and it lives in `dev_docs/`, `README.md`
or `CONTRIBUTING.md`. A fact copied into two files goes stale in one of them: an
earlier version of this file described a `lackey.cc` and a `symmetries.cc` that do
not exist, a GAP integration the README says was removed, a CMake version that was
wrong, and Boost.Program_options in place of cxxopts — all of it plausible, none of
it true. So this file routes rather than restates: what to read, the mistakes that
have actually been made in this tree, and what to run before committing.

## Read the relevant document first

A C++20 solver for subgraph isomorphism, maximum clique and maximum common subgraph:
backtracking search with constraint propagation, which can also emit a
VeriPB-checkable proof of its answer. That proof-logging ability shapes a lot of the
design and very little of it is guessable from the code alone. Read the document
covering what you are about to change *before* you change it.

| If you are... | Start with |
|---|---|
| getting oriented anywhere in the tree | [`dev_docs/architecture.md`](dev_docs/architecture.md) — the layer map, and what each component is for |
| touching a format reader, a writer, or `InputGraph` | [`dev_docs/file-formats.md`](dev_docs/file-formats.md) |
| writing or debugging proof logging | [`dev_docs/proof-logging.md`](dev_docs/proof-logging.md) — including which option combinations are supported, which is not all of them |
| touching an option, a filter, or the traits layer | [`dev_docs/option-compatibility.md`](dev_docs/option-compatibility.md) — which combinations are legal, which are silently disabled rather than refused, and what the sweeps cover |
| changing anything the solver does before search | [`dev_docs/preprocessor-refactor.md`](dev_docs/preprocessor-refactor.md) — where that code is going, which is not where it is |
| looking for what an option does, or a file format | [`README.md`](README.md) |
| about to commit | [`CONTRIBUTING.md`](CONTRIBUTING.md) |

`gss/*.hh` is the public API — `HomomorphismParams` and `solve_homomorphism_problem`
are the main entry point, with `clique.hh` and `common_subgraph.hh` alongside.
`gss/innards/` is everything that is not part of it. `src/` is the command-line
drivers, which use cxxopts.

## Mistakes that have been made here before

Each of these is a real bug or a real wasted afternoon, not a hypothetical. The
reasoning is in the linked document; the instruction is here so it is visible
without following the link.

- **Do not infer a graph-level property from how a graph happened to be built.**
  Adding a label to an undirected CSV file used to make it directed, because the
  labelled path went through `add_directed_edge`, which set the flag (#86, #87).
  Directedness is now declared to the `InputGraph` constructor and
  `add_directed_edge` requires it. See
  [`file-formats.md`](dev_docs/file-formats.md).
- **After changing what a reader reports, grep for everything that branches on it.**
  Fixing the above broke the unwritten invariant that every edge-labelled graph was
  also directed, and two callers had been relying on it: the model allocated the
  forward/reverse target rows only for directed patterns while the searcher always
  reads them once there are edge labels (a segfault), and the SIP decomposer
  discarded every edge label when rebuilding its reduced pattern (silently no
  solutions). Both were unreachable before and neither was caught by a type error.
- **Do not compare user-supplied label text against magic strings.** A filter
  skipping target edges labelled `"unlabelled"` outlived the representation it was
  guarding by five years, and turned a satisfiable instance into `status = false`
  for anyone who used that word as a label (#88).
- **Loops and the induced / locally-injective modes are where the bugs live.** Four
  separate ones: the self-loop adjacency term dropped from solution proofs (#49),
  supplemental-graph derivations not verifying on loopy graphs (#56),
  locally-injective enumeration over-pruning (#58), and the loop/clique shortcuts
  being wrong for induced mappings into loopy targets. When you touch propagation or
  a proof derivation, test a loopy instance and the induced and locally-injective
  combinations, not just the default path. `test-instances/small` and `large` have a
  self-loop for this reason.
- **Do not "tidy" the `FORCE` off the per-configuration flags in `CMakeLists.txt`.**
  CMake pre-creates `CMAKE_CXX_FLAGS_<CONFIG>` as empty cache entries during
  compiler detection in `project()`, so a plain `set(... CACHE ...)` there is a
  silent no-op and the build type gets none of its flags — which for `sanitize`
  means a build containing no sanitizers that still passes.
- **The empty `//` markers in the cxxopts option blocks in `src/` are
  load-bearing.** Each option block is one chained expression, and with
  `ColumnLimit: 0` clang-format packs it onto a single line hundreds of characters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ciaranm/glasgow-subgraph-solver](https://github.com/ciaranm/glasgow-subgraph-solver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
