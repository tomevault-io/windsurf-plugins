---
trigger: always_on
description: This repository is a reusable harness engineering starter kit. It teaches
---

# AGENTS.md

## Purpose

This repository is a reusable harness engineering starter kit. It teaches
agents how to add durable project rules, constraints, feedback loops, knowledge
storage, and drift checks to another repository.

When applying this kit to a target repository, the target repository is the
source of truth. Preserve its architecture, tools, package manager, docs, and
conventions.

## Core Rules

- Keep this kit prompt-first. Agents should inspect the target repository and
  adapt the smallest useful harness pieces instead of copying defaults blindly.
- Treat a target-local `./harness-starter-kit` clone as read-only reference
  material unless the user explicitly asks to edit the kit itself.
- Do not overwrite, delete, move, or re-clone target project files unless the
  user explicitly asks and the reason is clear.
- Prefer the target repository's existing docs, scripts, CI, package manager,
  tests, and naming conventions over starter-kit defaults.
- Make important rules enforceable where practical through lint, tests, type
  checks, import rules, CI, or drift checks. If automation is not practical,
  document the manual review point.
- Keep templates generic and conservative. Do not bake in a single product
  architecture.
- Record fixed user-visible runtime failures or high-risk bug paths that should
  not recur, including 5xx errors, crashes, security or permission bugs,
  data-loss risks, failed CI runs, failed harness checks, repeated agent
  mistakes, previously identified bug paths, or cross-environment mismatches in
  `docs/failures/*.md` unless the issue was purely transient or already covered
  by an existing failure note. If skipped, explain why in the final report.
- When adding or updating a failure note, name the regression test, fixture,
  smoke check, lint rule, drift check, CI gate, or manual review point that
  prevents or detects recurrence. If no check is practical, record why.

## Command Routing

- `/harness adopt`: use `commands/harness-adopt.md`. Apply prompt-first
  harness engineering to the target repository. Inspect first, preserve target
  source-of-truth, avoid blind template copying, and finish with an adoption
  report.
- `/harness doctor`: use `commands/harness-doctor.md`. It is diagnostic only:
  inspect and report, do not modify files, and do not remove a target-local
  `./harness-starter-kit` directory.
- `/harness update`: use `commands/harness-update.md`. Refresh the kit
  reference first, avoid blind overwrites, update `.harness/source.json` only
  after confirming the source, and finish with a Harness Update Report.
- `/harness refresh`: use `commands/harness-refresh.md`. Review stale or
  duplicated target harness guidance. Do not delete, archive, move, or rename
  files without explicit approval for the specific files.
- `/harness review sub-agent`: use `commands/harness-review.md` in sub-agent
  invocation mode. Treat the request as explicit permission to use a read-only
  reviewer subagent when available and permitted by the active runtime and tool
  instructions; if unavailable, blocked, not permitted, or failed, fall back to
  single-agent review and report the reason.
- `/harness review`: use `commands/harness-review.md`. Review the current
  change set from an opposing harness-engineering perspective. It is diagnostic
  by default and must not modify files unless the user explicitly asks to apply
  fixes after seeing the review.

## Project Analysis Rule

When asked to analyze, review, summarize, onboard to, or explain this project,
inspect these first when they exist:

- `README.md`
- `AGENTS.md`
- `.harness/source.json`
- `docs/theory/`
- `docs/decisions/`
- `docs/conventions/`
- `docs/domain/`
- `docs/failures/`
- `scripts/check_harness.py`
- `scripts/check_*.py`

Then summarize structure, current behavior, tests, documentation, known
decisions, known failures, drift checks, and recommended next work.

## Editing This Kit

- Read the relevant workflow doc before changing command behavior:
  `commands/harness-adopt.md`, `commands/harness-doctor.md`,
  `commands/harness-update.md`, `commands/harness-refresh.md`, or
  `commands/harness-review.md`.
- For adoption behavior, keep `docs/adoption-workflow.md`,
  `docs/prompts/apply-to-target-repo.md`, `docs/templates/adoption-report.md`,
  and examples aligned.
- For profile changes, keep `templates/profiles/<profile>/`,
  `docs/templates/profile-readme.md`,
  `docs/checklists/profile-maintenance.md`, fixture coverage, and
  `tests/test_profile_consistency.py` aligned.
- For drift script changes, keep the matching `templates/generic/scripts/`
  copy aligned when applicable.
- Favor clear Markdown and small Python scripts over heavyweight generators.
- Keep `examples/*-adoption-report.md` aligned with real adoption tests.

## Target Adoption Checklist

Use `docs/adoption-workflow.md` for the full procedure. The short version:

- Inspect the target repository before editing.
- Add or update only missing harness pieces: agent instructions, knowledge
  store, drift checks, CI/pre-commit wiring, or profile snippets when they fit.
- Use profile snippets as reference material, not mandatory transformations.
- Finish with an adoption report that lists changed files, checks run,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harnessworks/harness-starter-kit](https://github.com/harnessworks/harness-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
