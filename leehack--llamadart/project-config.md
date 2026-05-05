---
trigger: always_on
description: This file provides guidance for agentic coding assistants working in the llamadart repository.
---

# AGENTS.md

This file provides guidance for agentic coding assistants working in the llamadart repository.

## Build / Lint / Test Commands

### Development Commands
```bash
dart pub get                              # Install dependencies
dart format .                             # Format all Dart files
dart format --output=none --set-exit-if-changed .  # Check only, CI-friendly
dart analyze                              # Run static analysis/linting
dart analyze --fatal-infos              # Treat info-level lints as errors (CI)
```

### Testing Commands
```bash
dart test                                 # Run all platform-compatible tests (VM or browser)
dart test -p vm                           # Run only VM (native) tests
dart test -p chrome                       # Run only Chrome (web) tests
dart test --run-skipped -t local-only     # Run local-only E2E scenarios
dart test test/path/to/test_file.dart     # Run a single test file
dart test -p vm --coverage=coverage       # Run VM tests and collect coverage
dart pub global run coverage:format_coverage --lcov --in=coverage/test --out=coverage/lcov.info --report-on=lib --check-ignore
dart run tool/testing/check_lcov_threshold.dart coverage/lcov.info 70
```

### CI Standards
- `dart format --output=none --set-exit-if-changed .` checks formatting
- `dart analyze` runs the linter
- `dart test -p vm -j 1 --exclude-tags local-only` runs native tests sequentially (required for some OS)
- CI enforces >=70% line coverage for maintainable `lib/` code using `--check-ignore` (generated files marked with `// coverage:ignore-file` are excluded)

## Code Style Guidelines

### Imports
- Start with Dart SDK imports (`dart:core`, `dart:async`, etc.)
- Follow with package imports from external dependencies
- Use relative path imports for same-package files (`'../backends/backend.dart'`)
- Group imports with blank lines between categories
- No `show`/`hide` unless necessary for deconfliction

### Formatting
- Use `dart format` with default settings (no trailing comma, 80 character line length)
- Single blank line between top-level declarations
- Two blank lines between class-level sections

### Types & Declarations
- Explicit types on all public APIs: parameters, return types, fields
- Type inference (`var`, `final`) can be used for obvious local types
- Immutable data classes use `const` constructors where possible
- Private fields use leading underscore (`_modelHandle`)

### Naming Conventions
- Classes: `PascalCase` (e.g., `LlamaEngine`, `ChatSession`)
- Functions/methods: `camelCase` (e.g., `loadModel`, `setLogLevel`)
- Variables/params: `camelCase` with descriptive names
- Private members: leading underscore (`_isReady`)
- Constants: `lowerCamelCase` (e.g., `contextSize`, `gpuLayers`)
- Files: `snake_case.dart`
- Directories: `snake_case`

### Documentation
- All public members require Dart doc comments (`///`)
- Use triple-slash doc format with proper Markdown
- Include usage examples in class-level documentation
- Parameter and return types documented
- No TODO/FIXME comments in committed code

### Error Handling
- Use custom `LlamaException` hierarchy (defined in `lib/src/core/exceptions.dart`)
- Subtypes: `LlamaModelException`, `LlamaContextException`, `LlamaInferenceException`, `LlamaStateException`, `LlamaUnsupportedException`
- Accept optional `details` parameter for additional context
- Include human-readable message in `toString()`

### Library Structure
- Use `library;` directive in top-level export files
- Export clean public APIs via `lib/llamadart.dart`
- Keep implementation details in `lib/src/` subdirectories

### Platform Compatibility
- Use conditional imports for platform-specific backends (`if (dart.library.js_interop)`)
- Tag tests with `@TestOn('vm')` or `@TestOn('browser')`
- Keep `LlamaEngine` free of `dart:ffi` and `dart:io` for web support

### Architecture Principles
- Zero-Patch Strategy: Never patch upstream native sources in this repository
- Use wrappers and hooks for necessary integrations
- Modular separation: `engine/`, `backends/`, `models/`, `utils/`
- Abstract interfaces in `backends/backend.dart`

### Testing Standards
- New public APIs require unit or integration tests
- Test both Native (VM) and Web implementations for refactored shared logic
- Mark generated files with `// coverage:ignore-file` so coverage gates exclude them
- Use `expect` matchers over `assert`
- Close ports/streams in `setUp`/`tearDown` to avoid hanging
- Use `group` for logical test organization

### File Organization
- Library entry point: `lib/llamadart.dart`
- Public APIs in `lib/src/core/` with clear separation: `engine/`, `models/`, `template/`
- Tests mirror lib structure: `test/unit/` and `test/integration/`
- Native assets hook: `hook/build.dart` (downloads precompiled binaries)

### Const & Immutability
- Use `const` constructors wherever possible for immutable classes
- Data classes should have `const` constructors with `const` fields
- Factory constructors can be used but prefer `const` when feasible

### Async Patterns
- Use `Future<T>` and `Stream<T>` from `dart:async`
- Prefer async/await over chained `.then()` calls
- Use `StreamController` for custom streams with proper cleanup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leehack/llamadart](https://github.com/leehack/llamadart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
