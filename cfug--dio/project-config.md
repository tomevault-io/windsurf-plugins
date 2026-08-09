---
trigger: always_on
description: Language: English | [简体中文](AGENTS-ZH.md)
---

# Agent Contribution Guidelines

Language: English | [简体中文](AGENTS-ZH.md)

This document defines the rules for AI agents (and the humans operating them)
working on this repository — whether you are contributing a pull request or
assisting a maintainer locally. It supplements, and never overrides,
[CONTRIBUTING.md](CONTRIBUTING.md) and the
[Compatibility Policy](COMPATIBILITY_POLICY.md).

dio is one of the most depended-on packages in the Dart/Flutter ecosystem.
A single careless change can break tens of thousands of downstream projects.
Contributions here are not a playground: every change must be motivated,
tested, and compatible.

## 1. Motivation first — no speculative changes

**Do not invent work.** A change is only acceptable when it solves a problem
that actually exists.

- Every non-trivial change must be traceable to a concrete motivation:
  a reproducible bug, an accepted issue/discussion, an RFC-style proposal,
  or an explicit maintainer request. "This seems useful" is not a motivation.
- Before implementing a feature, answer these questions in the issue or the
  PR description — if you cannot, do not open the PR:
  1. What cannot be done (or is done poorly) with the current dio?
  2. Who needs this, and in which real-world scenario?
  3. Why does it belong in dio itself, instead of an interceptor, an adapter,
     a transformer, or a separate package? dio is intentionally extensible;
     most needs are served by its extension points without core changes.
  4. What is the cost — API surface, maintenance burden, compatibility risk?
- One PR, one concern. Do not bundle several unrelated features or fixes
  into a single PR. Bundled "improvement packs" might be closed unreviewed.
- For any user-facing feature, open an issue for discussion **before**
  writing code, unless a maintainer has already asked for it. A bare
  `Closes #NNNN` is not the same as prior discussion: the referenced issue
  must show that maintainers have expressed interest or accepted the
  direction. Feature PRs without that grounding waste both your tokens and
  the maintainers' time, and might be closed.

## 2. Tests are mandatory for logic changes

Every behavioral change must be proven by tests.

- Any change to logic requires new tests or adjustments to existing tests
  that fail without the change and pass with it. Bug fixes must include a
  regression test that reproduces the original report.
- CI reports coverage diffs on every PR. The published minimum threshold is
  low, but that is a floor, not a target: coverage of code you changed
  should not regress, and new logic (including error paths) should be
  covered by real assertions.
- Tests must be **effective and non-duplicated**:
  - Assert observable behavior, not implementation details.
  - Do not add tests that merely re-execute existing covered paths to
    inflate coverage numbers.
  - Search the existing suites first — extend an existing test group
    instead of creating a near-duplicate file.
- Put tests in the right place:
  - Package-specific behavior → `<package>/test/`.
  - Behavior that must hold across all adapters/platforms → the shared
    `dio_test` package.
- Run the checks locally before claiming they pass:

  ```bash
  melos run format   # or format:fix
  melos run analyze
  melos run test     # or targeted: test:vm / test:web / test:flutter
  ```

- Never state that tests pass without having run them. Never check a PR
  checklist item you have not actually done. Misreporting verification
  status may lead to the PR being closed.

## 3. Compatibility is sacred — avoid breaking changes

dio's public API is a contract with an enormous downstream. Treat every
public symbol as frozen unless a maintainer decides otherwise.

- **Default to non-breaking.** Prefer additive changes: new optional named
  parameters with safe defaults, new classes, new extension points.
- Do not change public method signatures, remove/rename public symbols,
  change default behavior, or alter thrown exception types without going
  through a deprecation cycle. Breaking changes belong in major releases.
  As dio's own [CHANGELOG](dio/CHANGELOG.md) preamble states, unavoidable
  breaking changes may occasionally ship in minor releases — those still
  require maintainer sign-off in advance and an entry in the
  [Migration Guide](dio/doc/migration_guide.md).
- If an API must go away, deprecate first and keep it working:

  ```dart
  @Deprecated('Use XXX instead. This will be removed in X.0.0')
  ```

  Deprecations must state their replacement and the removal version, and
  are only removed in the next major release, together with an entry in
  the Migration Guide. Target the *next* major, not a version beyond that.
- Do not raise the minimum Dart/Flutter SDK constraint of any package
  unless required by the [Compatibility Policy](COMPATIBILITY_POLICY.md)
  or its listed exceptions. CI tests against the minimum supported SDK;
  do not use language/library features beyond a package's lower bound.
- Watch for **behavioral** breaking changes too: changing defaults, header
  normalization, redirect/error semantics, or timing/ordering of
  interceptors can break downstream even when signatures are untouched.
- If a breaking change is genuinely unavoidable, stop and raise it in an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cfug/dio](https://github.com/cfug/dio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
