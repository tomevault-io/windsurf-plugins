---
trigger: always_on
description: - If behavior, CLI output/options, or configuration changes, update the relevant docs in `docs/` in the same change.
---

# Agent Instructions

- If behavior, CLI output/options, or configuration changes, update the relevant docs in `docs/` in the same change.
  Do not update `docs/` for minor changes.
- When asked to make a PR, first rebase your changes on latest `main`, then open a PR. Verify that the PR
  body is formatted correctly, then monitor CI and push fixes for any CI failures or merge conflicts.

## Changelog

- Any change with a user-visible effect must include an entry in `docs/changelog.md`.
- Add changelog entries under `## Unreleased` in `docs/changelog.md`.
- Changelog entries should be single bullets in plain language that explain the user-visible effect (not internal refactors).
- Bugfixes for bugs that were not in any release do not need changelog entries.

## Architecture

- Major files include:
  - `name_check_visitor.py` contains the core visitor that walks over code being type checked. It should visit
    every AST node exactly twice (in the collect and check phases). This file should
    contain the core checker logic, but delegate to other files (e.g., `attributes.py`,
    `type_object.py`, `stacked_scopes.py`) for narrower logic specific to those files.
  - `value.py` contains the core data structures representing types.
  - `relations.py` performs basic operations on types, such as subtyping, assignability, and intersections.
  - `type_object.py` contains rich data about classes.
  - `annotations.py` contains code for understanding annotations and typing-related objects and parsing them into pycroscope's internal
    data structures.
  - `arg_spec.py` parses callable objects and extracts their callable signatures.
  - `signature.py` contains representations of callable signatures and operations on them, such as type checking calls.
  - `checker.py` orchestrates type checking across multiple files and maintains caches that span across files.
  - `implementation.py` contains callbacks ("impls") that perform more precise checks or type inference on specific callables.
  - `stacked_scopes.py` contains logic for tracking which names are in scope and for local narrowing.
  - `attributes.py` retrieves attributes on objects.
- Techniques for keeping `name_check_visitor.py` clean include:
  - Collect short-lived data in attributes on the `NameCheckVisitor`, then aggregate it later. For example, to determine whether
    a function is a generator, set some marker when visiting `yield` expressions, and read the result at the end of visiting the
    `FunctionDef`. To handle nesting, stash any existing state when entering a new `FunctionDef`.
  - Make canonical internal representations (such as `Value` and `TypeObject` objects) rich enough to store the state you need;
    avoid creating additional complex objects internal to `name_check_visitor.py`.

## Testing

- Before finishing, run the linting/tests relevant to the files you changed.
- `test_self.py` is often useful for finding regressions. Run it with the `full` extra enabled or it will be skipped.
- Run conformance CI with the same interpreter you want pycroscope itself to use, because `tools/conformance_ci.py`
  invokes `sys.executable -m pycroscope` under the hood.
- Use this form from the repo root: `uv run --python 3.12 python tools/conformance_ci.py --typing-repo ~/py/typing`
  (optionally prefix `UV_CACHE_DIR=/tmp/uv-cache`).
- When fixing regressions found by `test_self.py`, add separate test cases instead of just relying on `test_self.py`.
- When writing test cases, always use code samples (`@assert_passes()`) instead of tests that directly
  invoke pycroscope functions. Code samples should represent user-written code that triggers the pycroscope
  feature under test and should not import internal pycroscope functions (except where needed for e.g. `assert_is_value`).
  Use direct unit tests (calling internal functions) only if testing a widely used API with a clear contract, such as `TypeObject.get_attribute` or `has_relation`.
- Prefer `assert_type()` over `assert_is_value()` for type assertions against types where possible; it's OK to keep
  `assert_is_value` for more complicated types that cannot be directly represented in user code.

## Coverage

- To reproduce the CI coverage run locally, use Python 3.14 and `pytest-cov`, for example:
  `UV_CACHE_DIR=/tmp/uv-cache uv run --python 3.14 --extra tests --extra asynq --extra codemod --with pytest-cov pytest --cov=pycroscope --cov-report=term-missing --cov-report=json:coverage.json pycroscope`
- To generate an HTML line-by-line coverage report locally, add `--cov-report=html` to the coverage command above.
  The report is written to `htmlcov/index.html`.
- The pinned-full-coverage check is checked in as `tools/check_full_coverage.py`; run it with
  `python tools/check_full_coverage.py coverage.json tools/full_coverage_files.txt`.

## Code Conventions

- Types and type signatures should have a single canonical internal representation,
  which is normalized when we parse annotations (e.g. in `annotations.py`). We should use
  the same representation regardless of whether an object was created in importable or
  unimportable mode.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JelleZijlstra/pycroscope](https://github.com/JelleZijlstra/pycroscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
