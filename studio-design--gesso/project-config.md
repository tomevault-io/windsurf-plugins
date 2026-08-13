---
trigger: always_on
description: provides endpoint/response coverage, schema-driven exploration, enum-drift and
---

# Repository guide for coding agents

This file is the shared source of repository-specific guidance. Keep it factual,
tool-agnostic, and aligned with the code and CI. Put agent-specific settings in
that agent's local configuration instead.

## Project scope

`studio-design/gesso` is a PHP 8.3+ test-time library for
validating requests and responses against OpenAPI 3.0, 3.1, and 3.2. Its core is
framework-agnostic; adapters support PHPUnit, Laravel, Symfony, and Pest. It also
provides endpoint/response coverage, schema-driven exploration, enum-drift and
strict-required checks, a spec doctor CLI, parallel coverage merging, and Laravel
route/spec parity.

Runtime dependencies are deliberately small. Framework, YAML, remote-reference,
faker, and Pest packages are optional; see `composer.json` and `docs/setup.md`.
Do not add a production dependency without first justifying the public and
maintenance cost.

## Repository map

- `src/OpenApiRequestValidator.php`, `src/OpenApiResponseValidator.php`: public
  framework-independent validation entry points.
- `src/Spec/`: spec loading, reference resolution, version/dialect selection,
  path matching, operation lookup, and schema conversion.
- `src/Validation/`: request/response validators, strict-required analysis, and
  shared validation boundaries.
- `src/Coverage/`, `src/PHPUnit/`: coverage state/renderers/merge protocol and the
  PHPUnit extension.
- `src/Psr7/`, `src/Laravel/`, `src/Symfony/`, `src/Pest/`: integration layers.
  Keep policy in the core when it is not framework-specific.
- `src/Fuzz/`, `src/Schema/`: generated exploration cases and enum-drift checks.
- `src/Internal/`: implementation details; do not expose these as public API.
- `bin/gesso`: Composer-installed CLI for `doctor` and `coverage:merge`.
- `tests/Unit`, `tests/Integration`, `tests/fixtures`: tests and representative
  OpenAPI fixtures. Pest integration tests run through a separate command.
- `tests/Integration/Conformance`: runs two upstream corpora, both pinned by
  commit SHA in `composer.json` — the official JSON Schema Test Suite through
  the schema converter (pinning the verdict delta), and the OpenAPI
  Initiative's example documents through the loader and `gesso doctor`. See
  `docs/conformance.md` before changing `OpenApiSchemaConverter` or the
  doctor's structural checks.
- `docs/`: focused guides. Start with `docs/setup.md`,
  `docs/supported-features.md`, `docs/coverage.md`, and `docs/versioning.md`.

## Architecture and invariants

The normal validation path is:

1. `OpenApiSpecLoader` decodes JSON or optional YAML, resolves internal and local
   file references, and caches named specs. HTTP(S) references are opt-in.
2. `OpenApiVersion` and `OpenApiSchemaDialect` select OpenAPI and JSON Schema
   behavior. OpenAPI 3.0 uses the Draft 07 compatibility pipeline; 3.1/3.2 retain
   native JSON Schema 2020-12 semantics where supported by opis.
3. `OpenApiPathMatcher` normalizes and matches request paths, then
   `OpenApiOperationResolver` selects the operation. OpenAPI 3.2 custom
   `additionalOperations` are case-sensitive; do not normalize them like fixed
   HTTP fields.
4. Request validation composes path, query, header, security, and body results.
   Response validation resolves status/content type and validates headers/body.
   Both return `OpenApiValidationResult` with Success, Failure, or Skipped.
5. Framework adapters turn results into test assertions and record observations
   in `OpenApiCoverageTracker`; the PHPUnit subscriber renders reports and gates.

Keep doctor diagnostics consistent with runtime validation: malformed spec nodes
must not pass one path and fail the other. Preserve discriminator enforcement and
the selected schema dialect when introducing alternate validation entry points.

Coverage is measured at `(method, path, status, content-type)` granularity. State
export/import and sidecar formats support parallel runs; treat versioned wire and
JSON output formats as compatibility surfaces.

Public symbols not marked `@internal`, CLI flags/exit codes, PHPUnit extension
parameters, versioned Laravel route-parity JSON output, warning category prefixes,
and documented wire formats are covered by the v1.x compatibility policy. Read
`docs/versioning.md` before changing any of them. Do not assert exact validator
error prose unless the wording itself is the contract.

## Setup and commands

```bash
composer install

# Narrow tests first
vendor/bin/phpunit tests/Unit/Spec/OpenApiSchemaConverterTest.php
vendor/bin/phpunit --filter test_method_name
vendor/bin/phpunit --testsuite Unit

# Repository checks
composer test
composer stan
composer cs-check
composer ci

# Dependency metadata checks when composer.json changes
composer validate --strict
composer audit --abandoned=fail

# Apply formatting (rewrites files)
composer cs
```

`composer ci` runs both PHP-CS-Fixer configurations, PHPStan, and PHPUnit. CI also
tests PHP 8.3-8.5 with PHPUnit 12-13, lowest dependencies, the optional Pest
integration and example, Composer validation/audit, and generated Markdown lint.
When changing one of those surfaces, run its focused check and rely on the matrix
for combinations unavailable locally. Pest tests require Pest 4 and PHPUnit 12;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [studio-design/gesso](https://github.com/studio-design/gesso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
