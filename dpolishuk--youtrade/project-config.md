---
trigger: always_on
description: This document governs how AI agents work on this Flutter project. We follow **Extreme Programming (XP)** as our primary development discipline. All work must align with XP values, principles, and practices while respecting Flutter/Dart conventions. Prefer the listed skills when applicable.
---

# AGENTS.md — Flutter Project Guidelines

This document governs how AI agents work on this Flutter project. We follow **Extreme Programming (XP)** as our primary development discipline. All work must align with XP values, principles, and practices while respecting Flutter/Dart conventions. Prefer the listed skills when applicable.

## XP Values (Non-Negotiable)

- **Communication**: Prefer clear, concise explanations and code that communicates intent. Face complexity with questions, not assumptions.
- **Simplicity**: Do the simplest thing that could possibly work. Avoid speculative design; solve today’s requirements, not tomorrow’s guesses.
- **Feedback**: Work in tiny cycles, run tests constantly, and validate assumptions early.
- **Courage**: Refactor mercilessly, delete dead code, and raise concerns about technical debt or process issues.
- **Respect**: Write code that others can read, maintain, and improve. Respect teammates’ time and the customer’s goals.

## XP Engineering Practices

### Test-First Development (TDD)

- **Always write a failing test before production code.**
- Cycle: write failing test → confirm it fails → write minimal code to pass → refactor → repeat.
- Prefer the `xpowers-test-driven-development` skill for every bug fix and feature.
- Keep tests fast, independent, and deterministic. Mock external dependencies; never test the real network or database in unit/widget tests.
- Run all tests before every commit. No commit should leave the suite red.

### Continuous Integration

- Integrate code frequently — multiple times per day.
- Keep the build under ten minutes: `flutter test`, `flutter analyze`, and `flutter format` must run quickly.
- Fix broken builds immediately; do not continue feature work on a red pipeline.

### Simple Design

- Follow the **Four Rules of Simple Design** (in priority order):
  1. Passes all tests.
  2. Reveals intention (clear names and structure).
  3. No duplication (DRY).
  4. Fewest elements (remove unused code, abstractions, and indirection).
- Avoid over-engineering; add abstraction only when duplication or pain demands it.
- Use the `xpowers-review-simplification` skill before introducing new patterns or abstractions.

### Refactoring / Design Improvement

- Refactor continuously, not in big batches.
- Refactoring is safe only with passing tests. Run tests after every small change.
- Use the `xpowers-refactoring-safely` skill for non-trivial refactors.
- Before large refactors, use `xpowers-refactoring-diagnosis` and `xpowers-refactoring-design`.

### Pair Programming

- Treat every change as if written in a pair: one person driving, one reviewing.
- As an AI pair partner, explain the “why” behind non-obvious decisions and invite alternatives.
- Rotate ideas and approaches; avoid heroics or single-person silos.

### Collective Code Ownership

- Any agent may improve any code anywhere in the project.
- Leave code cleaner than you found it.
- Do not tolerate “someone else’s code” as an excuse for poor quality.

### Coding Standard

- Follow the [Effective Dart](https://dart.dev/effective-dart) style guide.
- Prefer `final` and `const` where possible.
- Keep widgets small and composable; extract large build methods.
- Avoid nesting beyond 3–4 levels; extract private widgets or methods.
- Name files in `snake_case.dart` and classes in `PascalCase`.
- Use trailing commas for multi-line parameters and collections.
- Sort imports: Dart SDK, Flutter, third-party, project (relative last).
- Run `dart format --set-exit-if-changed lib test integration_test tool` before claiming work complete.

### Sustainable Pace

- Do not recommend death marches, heroic all-nighters, or shortcuts that create debt.
- Take small steps and keep the pace steady.

## XP Planning & Collaboration Practices

### Whole Team

- The team includes customer, developers, testers, and a coach when needed.
- Agents act as members of the whole team: ask clarifying questions, surface risks, and keep the customer’s goals visible.

### User Stories

- Work is expressed as small, user-visible stories.
- Stories are reminders for conversation, not specifications. Keep acceptance criteria concrete and testable.
- Break stories into the smallest slice that delivers value.

### Weekly / Quarterly Cycles

- Plan in weekly iterations (weekly cycle) aligned with quarterly goals (quarterly cycle).
- At the start of each week, know which stories are being delivered and what “done” looks like.
- Done means running, tested, integrated, and formatted — not “mostly done.”

### Small Releases

- Deliver working software in small, frequent increments.
- Prefer merging small, focused PRs over large batches.
- Each release/merge must leave `main` deployable.

### Slack

- Include low-priority cleanup or learning tasks in every plan that can be dropped if higher-priority work slips.
- This protects commitments and creates space for quality.

### Metaphor

- Maintain a shared system metaphor and consistent naming across the codebase.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dpolishuk/youtrade](https://github.com/dpolishuk/youtrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
