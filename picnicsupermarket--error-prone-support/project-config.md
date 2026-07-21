---
trigger: always_on
description: This file provides guidance to AI coding agents such as Claude Code, Google
---

# Agent Instructions

This file provides guidance to AI coding agents such as Claude Code, Google
Gemini, GitHub Copilot and OpenAI Codex when working with code in this
repository.

## Project overview

Error Prone Support is a Picnic-opinionated extension of Google's Error Prone
static analysis tool for Java. It provides additional `BugChecker`
implementations and Refaster rules that improve code quality, focusing on
maintainability, consistency, and avoidance of common pitfalls.

- **Maven Group ID**: `tech.picnic.error-prone-support`
- **License**: MIT
- **Docs**: https://error-prone.picnic.tech

See the [task-specific instructions](#task-specific-instructions) below and the
files in `.github/instructions/` for detailed conventions.
[CONTRIBUTING.md][contributing] provides additional context for human
contributors (bug reporting, PR process, module overview).

This project demands the highest quality bar. Favour correctness and precision
over speed. Do not cut corners, skip edge cases, or settle for "good enough".

## Build commands

Building requires **JDK 25** (use `sdk env` with SDKMAN). The project targets
JDK 21.

```sh
# Standard build and install.
mvn clean install

# Fast build skipping checks.
mvn clean install -Dverification.skip

# Build with non-fatal warnings.
mvn clean install -Dverification.warn

# Format code (google-java-format).
mvn fmt:format

# Full build: first against vanilla Error Prone, then against Picnic fork with
# self-check.
./run-full-build.sh

# Apply Error Prone auto-fixes to this project (also formats).
./apply-error-prone-suggestions.sh

# Run mutation tests on changed code (vs. upstream default branch).
./run-branch-mutation-tests.sh
```

### Running a single test

```sh
mvn test -pl error-prone-contrib -Dtest=DirectReturnTest -Dverification.skip
```

Replace the module (`-pl`) and test class (`-Dtest=`) as needed.

### Key Maven flags

- `-Dverification.skip`: disable non-essential plugins (fastest build).
- `-Dverification.warn`: make warnings non-fatal.
- `-Pself-check`: apply this project's checks against itself.

## Workflow

Implementation tasks must adhere to the following procedure:

1.  Create a solid plan before attempting implementation.
    * Tip: use `/using-superpowers` (Claude Code) or similar.
2.  Implement the desired changes and associated tests. When creating
    new files that must ultimately be committed, stage them immediately
    using `git add <file>`.
3.  Run the tests, and iterate until the tests pass.
4.  Commit your changes. Follow the conventions in
    [`.github/instructions/commit-message.instructions.md`][commit-message].
5.  Critically review your changes (both code and tests). Go through the
    applicable sections of
    [`.github/instructions/review.instructions.md`][review] and verify each
    item. Question whether each method, abstraction, and conditional is
    necessary. Simplify aggressively: eliminate redundancy, flatten unnecessary
    complexity, and reduce the code to its minimal correct form.
    * Use all code review skills available to you. Execute as adversarial
      subagents.
6.  Repeat the previous step, focusing on a different aspect each pass (e.g.
    control flow, then naming, then test coverage). Continue until diminishing
    returns.
7.  Commit your new changes (if any).
8.  Run a quick build using `mvn clean install -DskipTests
    -Dverification.skip`.
9.  Run `./run-branch-mutation-tests.sh` to determine mutation test coverage
    using Pitest (PIT) and try to resolve any surviving mutants listed in
    `<module>/target/pit-reports/mutations.csv`. See
    [`.github/instructions/testing.instructions.md`][testing] for guidance on
    interpreting results and documenting unkillable mutants.
10. If there are changes, commit them.
11. Run `./apply-error-prone-suggestions.sh` to clean up the code and inspect
    the resultant changes using `git diff`.
    - If this command fails, try to understand why and fix the issue.
    - If there are changes, validate that they make sense. If not, and they are
      due to newly introduced changes, undo the changes and attempt to fix the
      bug.
12. Once again, if there are changes, commit them.
13. Run `./run-full-build.sh` and attempt to resolve any build failures. Be
    efficient by attempting to resolve multiple warnings and errors in one go.
14. If applicable, commit changes once more.

## Skills

Skills in `.agents/skills/` provide focused guidance for specific task types.
Invoke them by name (e.g., `/write-java-code` in Claude Code) or let the agent
platform activate them automatically based on context.

**Implementation:** [`bug-checker`][skill-bug-checker],
[`refaster-rule`][skill-refaster-rule],
[`write-java-code`][skill-write-java-code],
[`write-documentation`][skill-write-documentation],
[`write-github-actions-workflow`][skill-write-github-actions-workflow],
[`write-shell-script`][skill-write-shell-script].

**Workflow:** [`write-commit-message`][skill-write-commit-message],
[`open-pull-request`][skill-open-pull-request],
[`upgrade-dependency`][skill-upgrade-dependency].

**Validation:** [`review-changes`][skill-review-changes],
[`run-mutation-tests`][skill-run-mutation-tests],
[`apply-self-check`][skill-apply-self-check],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PicnicSupermarket/error-prone-support](https://github.com/PicnicSupermarket/error-prone-support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
