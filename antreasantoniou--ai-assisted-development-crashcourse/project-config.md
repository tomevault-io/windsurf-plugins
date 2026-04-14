---
trigger: always_on
description: dart_coding_style_guidelines: |
---

dart_coding_style_guidelines: |
  1. **Language & Formatting**:
      - Use dart format --line-length 100 (configured in dart_tool/package_config.json) to
        guarantee consistent 2-space indentation and brace placement.
      - Run dart fix --apply and dart analyze on every commit; fail CI on warnings.
      - Include an analysis_options.yaml that extends the latest
        package:very_good_analysis/analysis_options.yaml (or flutter_lints for Flutter
        layers). Add project-specific rules only when they improve signal-to-noise.
      - Prefer named imports with show/hide; never use wildcard * imports.

  2. **Naming Conventions**:
      - **lowerCamelCase** for variables, functions, parameters, getters & setters.
      - **UpperCamelCase** for classes, enums, typedefs, extensions, mixins, and annotations.
      - **SCREAMING_SNAKE_CASE** for const values and enum entries that represent flags.
      - Prefix booleans with is, has, should, or can.
      - Avoid single-letter identifiers except trivial loop indices.

  3. **Documentation & Comments**:
      - Write DartDoc triple-slash /// comments for every public library, class,
        extension, mixin, and member. Include:
          * High-level purpose
          * Parameters ([paramName] — description)
          * Return value
          * Possible exceptions
      - Use inline // comments sparingly to explain why non-obvious code exists.
      - Keep comments in sync with behavior; outdated docs are worse than none.

  4. **Code Structure & Modularity**:
      - Embrace the Single Responsibility Principle; functions <40 LOC and classes
        <300 LOC are a smell that code is growing complex.
      - Organize related APIs into libraries (library directive) and export them via
        barrel files (<feature>/<feature>.dart exports).
      - Reach for part/part of only to split very large libraries; prefer packages
        otherwise.
      - Hide implementation details by prefixing internal identifiers with _.

  5. **Clarity, Intuitiveness & Simplicity**:
      - Leverage Dart's sound null-safety; favor final/const over var and
        late only when initialization is truly deferred.
      - Opt for extension methods, collection if/for, and pattern matching to make
        intent explicit.
      - Flatten deeply nested control flow by extracting private helpers.

  6. **Efficiency**:
      - Mark immutable objects const; they are canonicalized at compile-time.
      - Use isolates for CPU-bound tasks; avoid blocking the main isolate.
      - Cache heavy objects (e.g., regex, render objects) where reuse is expected.
      - Profile with DevTools before micro-optimizing; document benchmarks in /bench.

  7. **Testing**:
      - Write unit tests with the test package (or flutter_test for widgets).
      - Target ≥80 % line coverage on critical packages; gate PRs on coverage delta.
      - Use mocktail/mockito for fakes and golden_toolkit for widget-level
        regression tests.
      - Integration tests run via dart test -t integration or flutter test -d.

  8. **General Philosophy**:
      - Code for the next maintainer—clarity beats cleverness.
      - Follow the project's established idioms; when in doubt, justify deviations in
        the PR description.
      - Embrace code reviews, static analysis, and automated formatting as teaching
        tools.

  9. **CLI Output & Progress**:
      - For command-line apps, use package:dart_console or
        package:cli_progress to provide colored, width-adaptive output and progress
        bars.
      - Respect non-interactive environments: hide spinners behind
        if (stdout.supportsAnsiEscapes) checks.

 10. **Output Quality & Informativeness**:
      - Craft log messages with the logging package; route records through
        Logger.root.onRecord.listen to integrate with observability backends.
      - Include actionable guidance in error messages and surface stack traces only
        in verbose/debug modes.
      - Never import internal members directly; always go through the public
        package API.
      - Keep model identifiers centralized (e.g., lib/src/constants.dart) so upgrades
        require a single change.

### 11.2 Import Styles
- **Full Paths REQUIRED**: All `import` statements in Dart and Python MUST use the full package URI (e.g., `package:project_name/path/to/file.dart` or `import my_project.utils.helpers`). Relative paths (`../` or `./`) are strictly forbidden. This rule is non-negotiable.
- **Liberal Use of `show`**: For Dart imports, make liberal use of the `show` directive to explicitly declare which symbols are being imported. This prevents namespace pollution and improves clarity. Hiding symbols with `hide` is also acceptable where appropriate.

  Example:
  ```dart
  import 'package:guarded_core_openapi/api.dart' show Application, PrivacyEnum, UserProfile;
  ```

# Cursor AI ASSISTANT - SYSTEM PROMPT (.cursorrules)

## 1. INTRODUCTION
You are Cursor, an AI assistant designed to support the user, an expert in their field.

## 2. CORE PRINCIPLES
### 2.1. Principle of Minimal, Justified Edits
- **Surgical Precision**: All code modifications must be direct, necessary, and strictly scoped to the user's request. Avoid refactoring, renaming, or reformatting of unrelated code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AntreasAntoniou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
