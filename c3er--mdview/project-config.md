---
trigger: always_on
description: This repository is the legacy application that defines the user-visible behavior of Markdown Viewer. It is the ground truth for feature behavior and bugfix expectations. The code is older and more monolithic than the refactored successor, but its behavior is the standard to preserve.
---

# AGENTS.md

## Mission

This repository is the legacy application that defines the user-visible behavior of Markdown Viewer. It is the ground truth for feature behavior and bugfix expectations. The code is older and more monolithic than the refactored successor, but its behavior is the standard to preserve.

This file exists as guidance for AI-assisted maintenance and small feature work. It is intentionally conservative: do not rewrite the app; do targeted bugfixes and incremental improvements.

## Working style for this repo

- Treat the app as a stable, production-style tool, not a sandbox for large refactors.
- Prefer surgical fixes over architecture churn.
- Keep compatibility with Windows, macOS, and Linux behaviors.
- If you change behavior, update the relevant tests or add a focused regression test.
- Preserve the original UX and the minimal-scope character of the project.
- When the requirements or next steps are unclear, ask the user for clarification before making a consequential choice.
- When multiple approaches are plausible and none is clearly best according to the existing behavior, architecture, or history, ask the user to choose between them.

## AI operating rules

This repository is the behavioral reference and the maintenance base for Markdown Viewer. The history before `v2.0.0` and the later maintenance commits show a consistent pattern: keep the app understandable, fix the actual problem, and avoid broad churn unless the refactor clearly reduces long-term risk.

Agents in this repo should follow these rules:

1. Prefer root-cause fixes over surface-level patches.
2. Keep scope tight and user-visible behavior stable.
3. Preserve compatibility with file path handling, URL decoding, rendering behavior, keyboard interactions, and OS-specific edge cases.
4. Do not reject justified refactors. A larger cleanup is acceptable when it eliminates duplication, makes a bug fix trustworthy, or reduces the chance of regression.
5. When refactoring, do it in a way that keeps the architecture legible and testable, not in a way that merely makes the code look modern.
6. History matters: the app’s fragile areas are navigation, rendering, search, settings, and anything involving relative file paths or external resources.

This repo should still permit opportunistic refactors if the motivation is maintainability, not cosmetics. The goal is not to over-engineer, but to avoid low-quality, “cheapest possible” code that becomes impossible to extend safely.

## Documentation maintenance

Documentation is part of the implementation and must always describe the current state. Keep `README.md`, `CONTRIBUTING.md`, everything under `doc/`, `AGENTS.md`, and any skills or other AI guidance up to date. Any code change that makes a documented statement inaccurate must update that documentation in the same change. AI guidance is especially important: do not leave obsolete instructions, architecture descriptions, workflows, or feature claims behind.

The `CHANGELOG.md` has an additional maintenance rule: every user-facing change must be documented in its `## Current` section. Each new feature gets its own subsection; bug fixes go under a subsection named `Bugfixes`. If the session has a ticket reference, such as a GitHub issue, include that reference in the changelog entry. Keep everything outside `## Current` unchanged and treat released entries as historical record. Modify older release sections only when the user explicitly requests a specific change.

## Testing and validation

A good agent in this repo should include a small validation plan with each change, especially when the behavior is visual or interactive. This app is not only about passing unit tests; many realistic regressions are only visible in the UI.

Expected behavior:

1. Run the relevant automated tests first.
2. If the change touches file handling, rendering, navigation, search, or settings, provide a short manual smoke-test sequence.
3. Keep the manual steps concrete and easy to follow for a human operator.
4. If the change is behaviorally subtle, state what the reviewer should look for instead of only reporting that the code “should work.”

In short: do not assume that the AI is done after a passing test run. In this project, a brief test plan is often part of the fix.

## Version control

This project uses Git. Use read-only Git operations freely whenever the current code, its motivation, or the history of a decision is unclear. In particular, use commands such as `git log`, `git show`, `git blame`, and diffs to understand behavior, ownership, and the reason for existing code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c3er/mdview](https://github.com/c3er/mdview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
