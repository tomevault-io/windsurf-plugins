---
trigger: always_on
description: This file defines the working conventions for contributing to `{brickster}`. The goal is consistent API ergonomics, maintainable code, and tests that validate behavior (not just coverage).
---

# AGENTS.md

## Purpose
This file defines the working conventions for contributing to `{brickster}`. The goal is consistent API ergonomics, maintainable code, and tests that validate behavior (not just coverage).

## Rule Strength
- **Must**: required for all new/modified code unless explicitly agreed otherwise in review.
- **Should**: strong default; deviate only with clear, documented rationale.

## Product Intent
`{brickster}` is a pragmatic R toolkit for Databricks:
- thin, readable wrappers over Databricks REST APIs
- predictable function naming and return shapes
- first-class auth ergonomics (`PAT`, OAuth U2M, OAuth M2M)
- practical workflows (`DBI`/`dbplyr`, Unity Catalog volumes)

Prefer clear behavior over abstraction-heavy internals.

## Code Practices

### Function and file structure
- **Must** keep functions grouped by API domain in existing `R/*.R` files (clusters, jobs, volumes, auth, etc.).
- **Must** follow naming pattern: `db_<resource>_<action>`.
- **Should** add to an existing domain file before creating a new file.
- **Should** keep helper functions internal unless there is clear end-user value.

### Request wrapper pattern
- **Must** build requests via `db_request()`.
- **Must** execute JSON API requests via `db_perform_request()`.
- **Should** execute non-JSON/binary/header requests via `db_perform_response()` unless that helper clearly cannot support the endpoint.
- **Should** include in public API wrappers:
  - `host = db_host()`
  - `token = db_token()`
  - `perform_request = TRUE` when request inspection/testing is useful
- **Must** maintain stable return contracts:
  - `perform_request = FALSE` returns an `httr2_request`
  - action/delete endpoints default to `TRUE`/`FALSE`; return richer details only when meaningful endpoint data exists (for example: path/id/status details)
  - list/get endpoints should follow API response shape; return full bodies when pagination or metadata is needed and avoid dropping useful fields
  - explicitly document return shape differences in roxygen when endpoints in the same family differ

### Input validation and errors
- **Must** validate early.
- **Should** use `cli::cli_abort()` for user-facing argument errors when it can express the check clearly.
- **Should** use `stopifnot()` for compact multi-condition assertions or internal invariants where it is clearer.
- **Must** make error messages actionable and specific (what is wrong, and how to fix).
- **Should** avoid hidden side effects and stateful behavior unless required.

### Style and dependencies
- **Must** use `TRUE`/`FALSE` over `T`/`F` in new code.
- **Must** use `{fs}` for path manipulation and filesystem checks.
- **Must** use `{purrr}` for iteration in new/modified code (including tests); do not introduce new `*apply`/`vapply` iteration unless required for a specific external interface.
- **Should** keep comments sparse and only where behavior is non-obvious.

## Roxygen and Documentation
- **Must** use roxygen for all exported functions and run `devtools::document()` after API changes.
- **Must** include explicit `@returns` docs for every exported function, including `perform_request = TRUE` vs `FALSE` behavior when relevant.
- **Should** reuse parameter docs with `@inheritParams` (notably `auth_params` and shared request args).
- **Must** place functions in the correct `@family` for reference-site grouping.
- **Must** use `{lifecycle}` consistently for lifecycle changes:
  - add lifecycle badge in roxygen where relevant
  - emit deprecation warnings in each deprecated function (no hidden global helper if explicit per-function messaging is preferred)
- **Should** ensure new/changed public functions appear correctly in pkgdown reference structure (`_pkgdown.yml`) when needed.

## Test Strategy and Purpose
Tests are organized in three layers. Keep this structure.

### 1) Request-shape tests (`test-<domain>.R`)
- **Must** validate argument handling and request construction using `perform_request = FALSE`.
- **Must** check returned type (`httr2_request`) and key validation paths.

### 2) Offline helper tests (`test-<domain>-offline-helpers.R`)
- **Must** validate branching/business logic without network calls.
- **Must** use `local_mocked_bindings()` and assert behavior/output explicitly.

### 3) Integration tests (same domain test file as request-shape tests)
- **Must** guard real API tests with skip helpers (`skip_on_cran()`, auth/cloud guards).
- **Should** keep integration tests focused on key live behavior, not exhaustive API permutations.

### Test hygiene rules
- **Must** use `withr::local_envvar()` / `withr::local_options()` for temporary state.
- **Must not** use `options()` directly in tests.
- **Must not** use `Sys.unsetenv()` directly in tests.
- **Must not** use `brickster:::` in tests.
- **Must not** use `<<-`; track state with `new.env(parent = emptyenv())`.
- **Must** avoid nested mocking blocks.
- **Should** use at most one `local_mocked_bindings()` call per package within a single `test_that()` block; split phases into separate tests when needed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databrickslabs/brickster](https://github.com/databrickslabs/brickster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
