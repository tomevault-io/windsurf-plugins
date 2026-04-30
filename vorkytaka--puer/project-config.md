---
trigger: always_on
description: AGENTS — Agent instructions for building, linting, testing and style
---

AGENTS — Agent instructions for building, linting, testing and style

Purpose
- This file documents the commands and code-style expectations agents (automated tools and humans) should
  follow when operating on this monorepo. The repository is a Dart/Flutter workspace managed with
  Melos and uses a shared lint configuration in `packages/puer_lints/lib/puer_lints.yaml`.

Required tools
- Dart SDK >= 3.6.0 (see `pubspec.yaml` environment)
- Flutter SDK (this repo uses the Flutter version pinned in `.fvmrc`: 3.35.6)
- melos (used to run workspace-wide scripts)
- Recommended: fvm (to pin and use the Flutter/Dart versions defined in `.fvmrc`). Commands in this
  document prefer fvm-prefixed invocations for SDK-bound tooling to ensure consistency with CI.

Bootstrapping (install dependencies)
- From the repository root (preferred):

```bash
# fetch root dependencies and bootstrap the workspace using the pinned SDKs
fvm dart pub get
# install and link workspace packages, run pub get in each package
fvm dart pub run melos bootstrap

# alternatively, if melos is installed globally and you deliberately use the global install:
# melos bootstrap
```

Formatting & lint checks
- CI runs these steps (mirror locally before creating PRs):

```bash
# format all packages (CI: fails if there are changes)
fvm dart pub run melos format --set-exit-if-changed .

# full format (modifies files)
fvm dart pub run melos format .

# run analyzer (Flutter-aware analysis used in CI)
fvm flutter analyze .
```

Notes:
- Prefer using `melos format` for workspace-level formatting so configuration is applied uniformly. When
  running locally via the pinned SDKs, use `fvm dart pub run melos format`.
- If you use FVM, run SDK-bound commands with `fvm dart` and `fvm flutter` (examples above). If you
  intentionally rely on a globally installed `melos`, it's acceptable to run `melos ...` directly — but
  prefer the `fvm dart pub run melos ...` form to ensure the correct Dart runtime.

Running tests
- Run all package tests (CI command):

```bash
# runs the `test` script defined in the root pubspec via melos
fvm dart pub run melos run test
```

- Run tests only for packages that have tests (melos exec alternative used in repo):

```bash
fvm dart pub run melos exec --dir-exists=test -- "fvm flutter test"
```

- Run tests for a single package (examples):

```bash
# pure Dart package
cd packages/puer
fvm dart test

# Flutter package
cd packages/puer_flutter
fvm flutter test
```

- Run a single test file (from the package root):

```bash
# Dart
cd packages/puer
fvm dart test test/feature/feature_test.dart

# Flutter
cd packages/puer_flutter
fvm flutter test test/feature/feature_provider_test.dart
```

- Run a single test by name (filter):

```bash
# pattern matches test names
fvm dart test --name "should emit state"            # from Dart package
fvm flutter test --name "should emit state"         # from Flutter package
```

- Run a single test using melos (scoped):

```bash
fvm dart pub run melos exec --scope=puer -- "fvm dart test test/feature/feature_test.dart"
# or for Flutter package scope
fvm dart pub run melos exec --scope=puer_flutter -- "fvm flutter test test/feature/.."
```

CI (what the pipeline runs)
- See `.github/workflows/validate_repository.yml` for the exact sequence: extract Flutter version from
  `.fvmrc`, install Flutter, run `fvm dart pub run melos format --set-exit-if-changed .`, `fvm flutter analyze .`, and
  `fvm dart pub run melos run test`.

Coding style overview (high-level)
- This monorepo enforces a shared lint set. Each package includes `include: package:puer_lints/puer_lints.yaml`
  (see `packages/puer_lints/lib/puer_lints.yaml`) which is the authoritative source for rules.
- Agents should follow these conventions (summary of the most impactful rules):

- Types & declarations
  - Always declare return types on public functions and methods (`always_declare_return_types: true`).
  - Type-annotate public APIs (`type_annotate_public_apis: true`). Prefer explicit types rather than `dynamic`.
  - Prefer `final` (and `const` where applicable) for local variables and fields (`prefer_final_locals`,
    `prefer_final_fields`, `prefer_const_literals_to_create_immutables`).

- Imports
  - Follow `directives_ordering`: group imports in this order: `dart:` first, then `package:`, then relative
    imports. Keep a blank line between groups.
  - Prefer relative imports for files inside the same package (`prefer_relative_imports: true`).

- Formatting
  - Use the repo formatter via the pinned SDK: `fvm dart format` or the melos wrapper `fvm dart pub run melos format`.
  - Prefer single quotes for strings (`prefer_single_quotes: true`) unless the string contains single quotes.
  - Trailing commas are required on multi-line constructors/collections to produce stable formatting
    (`require_trailing_commas: true` and `formatter.trailing_commas: preserve`).

- Naming
  - File names: snake_case (enforced by `file_names: true`).
  - Types (classes, enums, typedefs): PascalCase / UpperCamelCase (`camel_case_types: true`).
  - Variables, functions, parameters: lowerCamelCase.
  - Private members: prefix with `_`.

- Error handling and exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vorkytaka/puer](https://github.com/Vorkytaka/puer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
