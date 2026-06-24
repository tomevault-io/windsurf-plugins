---
trigger: always_on
description: Incan is a Python-like language that compiles to Rust. The compiler itself is written in Rust and generates native Rust code via an IR-based pipeline. This document contains guidance for AI agents working on the codebase.
---

# Agent Instructions for Incan Development

Incan is a Python-like language that compiles to Rust. The compiler itself is written in Rust and generates native Rust code via an IR-based pipeline. This document contains guidance for AI agents working on the codebase.

> **CRITICAL — NO `.unwrap()` / `.expect()` ANYWHERE.** This is the single most important rule.
> Multiple modules enforce `#![deny(clippy::unwrap_used)]` and `#![deny(clippy::expect_used)]`.
> This applies to **all** code — production, tests, examples. No exceptions, no shortcuts.
> Use `?` with `Result`-returning test functions, or propagate errors explicitly.
> See [Error handling in tests](#error-handling-in-tests) for the correct pattern.

> **CRITICAL — THE USER DECIDES WHAT IS RELEVANT.** Scope, PR boundaries, and which files “belong” on a branch are **the maintainer’s call**, not the agent’s. Never label work as “unrelated PR noise,” “cleanup,” or “hygiene” as a reason to remove or revert it. Always check with the user when in doubt.
>
> **FORBIDDEN without explicit user approval that quotes the exact paths or commands:** anything that overwrites or deletes uncommitted work — including `git checkout -- <path>`, `git restore <path>`, `git clean`, `git reset --hard`, `stash drop`, or equivalent. If you believe files should be split, reverted, or left out of a PR, **state that and ask**; do not run destructive git operations on your own initiative.
>
> **Commits and pushes:** The maintainer commits code unless they **explicitly** ask you to run `git commit` or `git push`. Implement and test in the working tree; offer a suggested commit message as text. The `/start-work` skill states the same rule.

## Key References

| Document                 | Path                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------ |
| Rust coding conventions  | [`workspaces/docs-site/docs/contributing/explanation/readable-maintainable-rust.md`] |
| Project architecture     | [`workspaces/docs-site/docs/contributing/explanation/architecture.md`]               |
| Layer boundaries         | [`workspaces/docs-site/docs/contributing/explanation/layering.md`]                   |
| Writing RFCs             | [`workspaces/docs-site/docs/contributing/how-to/writing_rfcs.md`]                    |
| Contributor guide        | [`CONTRIBUTING.md`]                                                                  |
| GitHub issue templates   | [`.github/ISSUE_TEMPLATE/`]                                                          |
| Implementation learnings | [`.agents/learnings.md`]                                                             |

[`workspaces/docs-site/docs/contributing/explanation/readable-maintainable-rust.md`]: workspaces/docs-site/docs/contributing/explanation/readable-maintainable-rust.md
[`workspaces/docs-site/docs/contributing/explanation/architecture.md`]: workspaces/docs-site/docs/contributing/explanation/architecture.md
[`workspaces/docs-site/docs/contributing/explanation/layering.md`]: workspaces/docs-site/docs/contributing/explanation/layering.md
[`workspaces/docs-site/docs/contributing/how-to/writing_rfcs.md`]: workspaces/docs-site/docs/contributing/how-to/writing_rfcs.md
[`CONTRIBUTING.md`]: CONTRIBUTING.md
[`.github/ISSUE_TEMPLATE/`]: .github/ISSUE_TEMPLATE/
[`.agents/learnings.md`]: .agents/learnings.md

Skills, learnings, and agent notes live under **this repository’s** `.agents/` directory (committed here).

## General Workflow

1. **Branch from main**: Create a feature branch using the naming convention `<type>/<issue>-<slug>`, where type is `feature`, `chore`, or `bugfix`. Examples: `feature/165-implement-rfc-031-library-system-phase-1`, `chore/88-vocab-drift-guardrails`, `bugfix/42-fix-parser-crash`. Use the `/start-work` skill to automate this.
2. **Follow RFCs**: RFCs in `workspaces/docs-site/docs/RFCs/` are the spec — implement exactly what they say.
3. **Run tests**: `make test` must pass before considering work complete. Run targeted tests during development; run the full suite when you finish.
4. **Update snapshots**: `INSTA_UPDATE=1 cargo test --test codegen_snapshot_tests` to update changed snapshots.
5. **Boy Scout Rule**: Leave every file you touch in better shape than you found it — fix stale TODOs, missing doc comments, unused imports, misleading names.
6. **Documentation gate (mandatory)**: Before finalizing any change, audit every touched Rust module and ensure rustdocs are present and accurate for all new/changed functions and methods in changed Rust source files. This is enforced mechanically by `scripts/check_changed_rustdocs.py` through `make pre-commit-fast` and `make pre-commit`.

### Pattern intake before edits

Before touching production code, tests, or docs for a non-trivial change, do a short pattern intake:

- Identify the active area: parser, typechecker, lowering, emission, stdlib, CLI/tooling, docs, or tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [encero-systems/incan](https://github.com/encero-systems/incan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
