---
trigger: always_on
description: Use when: user mentions coverage, lint, linter, static analysis, code quality checks, 100% coverage, coverage gate, lint baseline, test report, TEST-REPORT.md, or 'is the build clean'; BEFORE declaring work done, summarizing what you built, or saying 'all passing/working/done/clean'; BEFORE committing or pushing; completing any feature/bugfix/refactor in a project that tracks test coverage OR has linters/analyzers configured (ruff, eslint, mypy, clang-tidy, jbinspect/inspectcode, golangci-lint, 
---


# Maintaining Full Coverage

## Overview

If the coverage report doesn't say 100% - or the linter has findings - you're not done. *(Unless the project never reached that bar and your current task isn't to get it there. Which of those you're in changes what "done" means - see Three Modes below.)*

**Core principle:** Every line of production code must be (a) exercised by a test and (b) clean against every linter/analyzer the project has configured. Uncovered lines are either untested (write a test) or unreachable (delete them). Lint findings are either real (fix them, ideally by restructuring) or genuine false positives (suppress per-case with explicit approval).

**Tests are not the only validators.** "Coverage" here means *covering the code with every check the project has* - tests for behavior, linters and analyzers for structure, type checkers for types. They're the same shape in the completion gate: machine-verifiable checked properties that must report clean before you declare done. Run them all, gate on all of them, restructure rather than suppress.

**Why this matters most:** Tests and linters earn their keep when they *accidentally surface real bugs*. A "unused variable" warning can reveal a typo that broke a code path. A `nullable` analyzer flagging a deref can pinpoint a real crash you missed. An uncovered branch can mean the condition is unreachable - i.e. dead code, often a bug. The discipline isn't paperwork; it's a structured way to make latent bugs visible. Treat findings as evidence first, noise second.

**This means ALL code, in ALL languages, in the ENTIRE repo.** A C# project with a C++ native library needs 100% coverage in both C# and C++, and Roslyn analyzers on the C# AND clang-tidy on the C++. A Python backend with a JavaScript frontend needs coverage.py + istanbul/c8 AND ruff + eslint. If production code exists in the repo and it's in a language you can compile/run, it needs coverage tooling, lint tooling, and tests. No language gets a pass.

**Violating the letter of this rule is violating the spirit of this rule.**

This skill is the final layer in a three-skill stack:

1. `test-driven-development` - writes tests before code
2. `verification-before-completion` - proves tests pass with evidence
3. `maintaining-full-coverage` - proves every line is covered AND every analyzer reports clean, and the report is updated

TDD is upstream discipline. Verification is evidence. This skill is the metric gate. When several completion skills are in play, the order is: this skill's gate (tests / lint / coverage) -> smoke-test -> docs-update -> declare done / commit.

## Three Modes: what "the bar" means here

The 100%-coverage / 0-findings bar is the destination. Whether *this task* must arrive there depends on where the project starts and what you were asked to do. **Read the project's current state first** - the checked-in `TEST-REPORT.md` (if any) plus a fresh coverage + lint run - then place yourself in one of three modes:

**1. Maintain - the project is already clean.** Coverage is 100% and every configured linter reports 0 findings (the report says so and a fresh run confirms it). The bar is absolute: hold it. Your change must not drop coverage below 100% and must not add a single finding. This is the strict gate - a regression here is blocked, full stop. Everything below about the completion gate and escalation ladder applies at full strength.

**2. Close the gap - reaching the bar IS the task.** The project isn't at 100% / 0 findings, and the user asked you to get it there ("add coverage", "clean up the lint", "get this to green", "set up the gate"). Then 100% / 0 is the deliverable and the full escalation ladder applies to every uncovered line and every finding - same strict bar as Maintain, because reaching it is the point.

**3. Best effort - the project is dirty and you're doing something else.** The project is below 100% / has findings, and your task is a feature, bugfix, or refactor - not a coverage/lint cleanup. Demanding the *whole repo* reach 100% before you can finish an unrelated feature is the hardline mistake this mode exists to prevent. The bar here is a **ratchet, not an absolute**:

- **Cover and clean what you touch.** New and changed production code gets tests and is lint-clean. You don't get to add debt just because debt already exists.
- **Don't regress the baseline.** Coverage percentage must not fall and the finding count must not rise versus the checked-in report. Those numbers are the floor.
- **Surface pre-existing debt; don't silently inherit it.** The 1,244 findings you didn't create aren't a blocker for *this* task - but record them in the report as the baseline and flag them (suggest a cleanup task). Don't pretend they're absent, and don't let them quietly grow.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtschoen/skills-maintaining-full-coverage](https://github.com/mtschoen/skills-maintaining-full-coverage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
