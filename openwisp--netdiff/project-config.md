---
trigger: always_on
description: `netdiff` is a Python library for parsing network topology data and comparing network graph snapshots.
---

# AGENTS.md

## Project Overview

`netdiff` is a Python library for parsing network topology data and comparing network graph snapshots.

Core code lives in `netdiff/`:

- `parsers/` contains parsers for supported topology formats and protocols.
- `utils.py`, `exceptions.py`, and `info.py` provide shared graph and metadata helpers.
- Tests live in `tests/` and `netdiff/tests.py`.

## Source of Truth

- Use `README.rst` for setup, package usage, and baseline test commands.
- Use `.github/workflows/ci.yml` for CI-tested dependencies, QA/test commands, env vars, and supported Python versions.
- Use GitHub issue/PR templates when asked to open issues or PRs.

If instructions conflict, repository config and CI workflows win first, docs next, and this file is supplemental.

## Contributing Guidelines

- Before editing, inspect the relevant implementation, tests, documentation, and configuration. Follow existing repository patterns and do not invent behavior or requirements.
- Keep each contribution focused and change only the lines necessary for its goal. Do not include unrelated refactors, formatting churn, or generated and dependency-file changes unless explicitly required.
- Add or update focused tests for every behavior change. Use test-driven development when the scope is very clear, such as bug fixes or narrowly scoped changes. For new features, tests may be added after implementation, but confirm they fail when key feature code is removed. When a test failure does not clearly state the expected outcome that was not met, add an explicit assertion message.
- Run `openwisp-qa-format` after each change when available.
- Run the relevant targeted tests, builds, and documented QA checks, including `./run-qa-checks` when provided. Do not claim a change is complete when verification fails; report the failure or blocker.
- When requirements, intended behavior, or an unexpected failure are unclear, stop and seek clarification instead of making speculative changes.
- When starting work on a new issue, create a new branch from `master`. Use `issues/<issue-number>-<short-title>` for issue work; otherwise, use a short, descriptive branch name.
- Commit messages must be descriptive and use past tense. Past tense is a writing guideline that agents and contributors must follow; it is not checked automatically. For issue work, use an allowed prefix and a capitalized, past-tense subject ending with `#<issue-number>`, for example `[fix] Fixed perennial "modified" state #213`. Repeat the issue reference in the body with `Fixes`, `Closes`, `Resolves`, or `Related to` as appropriate. After creating a commit, use `openwisp-commit --check` to validate the current `HEAD`; it cannot validate a proposed message. Use `openwisp-commit --check --rev-range <range>` for an existing commit range, and `cz -n cz_openwisp info` to view allowed prefixes and message structure.
- Add an explanatory commit body only for substantial changes, new features, or non-obvious bug fixes. The releaser automatically publishes the subject of `[feature]`, `[change]`, `[change!]`, `[deps]`, and `[fix]` commits, including scoped variants, in the changelog. Write those subjects in clear, user-friendly language suitable for release notes.
- Send new commits in response to review feedback instead of amending existing commits.

## Development Rules

- Follow the DRY principle: do not duplicate information or code across files.
- Respect module boundaries and encapsulation. The module that owns a model, stored state, lifecycle, or domain invariant must expose the cohesive public operation that reads or changes it. Integrations must use that operation, not write its fields, coordinate multi-step changes to its internal state, or depend on its storage representation. Prefer behavior-oriented public APIs over setters for internal flags. When an integration needs a missing capability, add it to the owning module with invariant tests, then call it from the integration.
- Preserve public APIs, parser outputs, graph comparison semantics, and supported input formats unless explicitly required.
- Process large input files, API responses, and generated output incrementally or in bounded batches when the operation permits it.
- Do not create unnecessary full copies of a large collection in memory.
- When a parser or algorithm requires the complete input in memory, avoid additional full-size intermediate collections unless they are necessary.
- When consuming a paginated remote API, process one page at a time instead of collecting every page before doing work.
- Place imports at the top of the file. Only defer imports when necessary (e.g., Django model imports inside functions or methods where the app registry is not yet ready).
- Avoid unnecessary blank lines inside function and method bodies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwisp/netdiff](https://github.com/openwisp/netdiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
