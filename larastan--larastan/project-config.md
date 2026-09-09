---
trigger: always_on
description: Larastan adds Laravel-aware analysis to PHPStan. Find the nearest existing
---

# Larastan contributor guidance

Larastan adds Laravel-aware analysis to PHPStan. Find the nearest existing
implementation and follow its complete pattern before adding a new one.

## Sources of truth

- `composer.json` defines supported PHP, Laravel, PHPStan, and PHPUnit versions.
- `extension.neon` defines services, extension tags, parameters, and defaults.
- `CONTRIBUTING.md` documents setup and contribution expectations.
- `.github/workflows/` defines the complete CI matrix.

Do not copy version numbers or CI matrices into documentation. Link to their
source instead so guidance does not drift.

## Where a change belongs

### For source code
- Dynamic return types belong in `src/ReturnTypes/`.
- Method and property discovery extensions belong in `src/Methods/` and
  `src/Properties/`; reusable PHPStan reflection implementations belong in
  `src/Reflection/`.
- Custom PHPStan types and type-node resolvers belong in `src/Types/`.
- Diagnostics belong in `src/Rules/`; cross-file data belongs in
  `src/Collectors/`.
- PHPStan stubs belong in `stubs/`.

### For tests
- Isolated behavior belongs in `tests/Unit/`, reflection extension behavior in
  `tests/Reflection/`, inline type inference in `tests/Type/`, rule diagnostics
  in `tests/Rules/`, and full-file analysis in `tests/Integration/`.
- Shared application classes, configuration, migrations, views, and providers
  belong in `tests/application/`.
- External-application compatibility belongs in `e2e/`.

Keep related implementation, registration, tests, and documentation together.
Check sibling extensions for the full set of files a change requires.

## Before changing behavior

- Reproduce a bug with the smallest test that fails before the fix.
- When existing behavior looks deliberate or version-dependent, inspect git
  history and the relevant Laravel and PHPStan source, tests, PRs, or issues
  before replacing it.
- Establish framework semantics from upstream tests or a focused runtime probe
  before encoding them as PHPStan types.
- Before creating an extension, check whether PHPDoc in stubs, including
  generics, conditional return types, and assertions, can express the behavior
  clearly and accurately. Prefer stubs when they can; otherwise use the smallest
  dedicated extension point that fixes the root cause. Avoid speculative
  helpers, generic machinery, and broad refactors.
- Keep unrelated discoveries out of the current change; report or fix them
  separately.
- If a test describes correct runtime behavior, fix Larastan's implementation,
  stub, or declaration instead of weakening the assertion.
- Keep inferred types both sound and useful on realistic Laravel code. Do not
  broaden a type merely to satisfy a test or narrow it merely to hide errors.
- Do not silence new failures with the baseline or ignore rules.

## Extension wiring and contracts

- Register extensions in `extension.neon` with the tag required by their
  PHPStan interface. For feature-gated tags or explicit `active` arguments,
  follow the nearest existing registration pattern.
- Add every Larastan-owned public configuration parameter to both `parameters`
  and `parametersSchema`, then document it in
  `docs/custom-config-parameters.md` or, for rule-specific settings, alongside
  the rule in `docs/rules.md`.
- New diagnostic identifiers use the `larastan.` prefix and dotted camelCase
  segments, such as `larastan.console.undefinedArgument`.
  `rules.modelAppends` is a legacy outlier; do not copy it.
- Document user-visible rules in `docs/rules.md`, custom types in
  `docs/custom-types.md`, and other features in `docs/features.md`.
- Reflection extensions that implement `hasMethod()`/`getMethod()` or
  `hasProperty()`/`getProperty()` must make both calls agree. When `has*()` does
  runtime-, container-, or resolution-dependent discovery, cache the found
  reflection under the same stable key for `get*()`; deterministic extensions
  may reconstruct it.
- For dynamic return type extensions, return `null` when the extension cannot
  refine the call. For other extension types, follow their documented fallback
  semantics. Use `ErrorType` only for a genuinely invalid expression, following
  the nearest sibling.
- PHPStan extension code that resolves services from the booted Laravel
  application should use the existing `HasContainer` mechanism. Its `resolve()`
  method deliberately returns `null` when the application cannot resolve a
  service; preserve that fallback.

## Stub files and Laravel versions

- Shared stubs live in `stubs/common/`; version-specific overrides live in the
  numbered directories.
- Stub discovery keeps only the newest applicable file for each relative path.
  A versioned file therefore replaces the whole earlier file; it is not merged
  declaration by declaration.
- Put a signature in the earliest version where it is valid and add a versioned
  override only where the signature changes.
- Keep version-specific tests at the same boundary as the corresponding stub or
  framework behavior. Use `laravel_version_compare()` like nearby tests.

## PHPStan type handling

- When identifying a type, prefer its query methods or `isSuperTypeOf()`. Use
  `accepts()` for assignment or argument compatibility. Use `instanceof` only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [larastan/larastan](https://github.com/larastan/larastan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
