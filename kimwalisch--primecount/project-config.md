---
trigger: always_on
description: These instructions apply throughout the primecount repository.
---

# Project instructions

These instructions apply throughout the primecount repository.

## Project overview

primecount is a command-line program and C/C++ library for computing π(x), the number of primes less than or equal to x. It uses highly optimized combinatorial prime-counting algorithms with OpenMP parallelization. Gourdon's algorithm is the default.

Read `README.md` once at the beginning of each new session before making code changes to understand the project's capabilities, usage, and algorithm overview. Consult relevant sections again as needed. Use the build and verification instructions in this `AGENTS.md` when developing.

## Core algorithms and required reading

primecount implements a family of combinatorial prime-counting algorithms. The Gourdon algorithm in `src/gourdon` improves on the Deléglise-Rivat algorithm in `src/deleglise-rivat`, which improves on the Lagarias-Miller-Odlyzko (LMO) algorithm in `src/lmo`.

These algorithms combine many formulas. Two of the most important contributions are the easy special leaves and the hard special leaves. Read the related mathematical papers before editing core algorithm code. Use the following mappings:

| Component | Implementation files | Required papers |
| --- | --- | --- |
| Easy special leaves | `src/deleglise-rivat/S2_easy.cpp`, `src/gourdon/AC.cpp`, `src/gourdon/AC_*.hpp` | `doc/Easy-Special-Leaves.pdf` |
| Hard special leaves | `src/deleglise-rivat/S2_hard.cpp`, `src/gourdon/D.cpp`, `src/gourdon/D_*.hpp` | `doc/Hard-Special-Leaves.pdf`, `doc/Hard-Special-Leaves-SIMD-Filtering.pdf` |
| Partial sieve function | `src/phi.cpp`, `src/phi_vector.hpp`, `src/phi_vector.cpp` | `doc/Partial-Sieve-Function.pdf` |

For other core algorithm code, identify and read the related mathematical paper before editing, using references in the source files and project documentation.

Read each relevant paper once per Codex session/chat, when work on the corresponding algorithm first begins and before the first code edit. For example, work on `src/gourdon/D.cpp` requires reading both `doc/Hard-Special-Leaves.pdf` and `doc/Hard-Special-Leaves-SIMD-Filtering.pdf`. Reuse that reading for subsequent edits in the same session; do not reread a paper for every edit or message. When work moves to another component, read any required papers that have not yet been read in the session. In a new session/chat, read the relevant papers again before editing.

Use the papers to understand the mathematics, but do not blindly copy their notation into variable names. Choose names that make the source code readable and fit existing naming conventions. Prefer shorter names when they remain clear.

## Vendored libraries

Exclude these vendored libraries from changes to the primecount codebase, including requests to change all occurrences of a pattern:

- primesieve: `lib/primesieve`
- libdivide: `include/libdivide.h`

Fixes to these libraries belong in their upstream repositories. For primesieve, update the vendored copy afterwards using `scripts/update_libprimesieve.sh`. The user usually performs this update manually; leave it to them unless they ask you to perform it.

## Preserve user changes

Preserve existing user changes, including staged changes. Do not revert, overwrite, or recreate code the user intentionally changed or removed unless the requested task requires it.

## Compatibility

Keep primecount's implementation compatible with C++14 and preserve existing platform support. C++14 is only a requirement for building primecount itself: applications using its public C++ headers and linking against the library must continue to work with C++11 or later. Keep public headers compatible with C++11 and avoid propagating a C++14 requirement to library consumers through build or package metadata. Do not introduce newer language requirements, dependencies, or public API changes unless the task calls for them.

## Coding conventions

primecount has no official coding style guide that can be enforced by a tool. Infer the coding style from the file being edited and follow its existing formatting. If the file is too small or lacks examples of the code construct being written, inspect one or two other source files to determine how to format it.

Keep changes limited to the requested task. Avoid unrelated refactoring, renaming, formatting, or whitespace changes. Preserve the existing file encoding and line endings.

- Do not break a variable initialization immediately after `=` except in rare cases, such as a complex boolean initializer with many conditions.
- Prefer a multi-line `if` condition over introducing a boolean variable used only for that condition.
- Split compound preprocessor conditions in `#if` and `#elif` directives across multiple lines, with one condition per line. Use `\` line continuations and align the continued conditions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimwalisch/primecount](https://github.com/kimwalisch/primecount) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
