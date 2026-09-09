---
trigger: always_on
description: `ansible-openwisp2` is the official Ansible role for installing and upgrading OpenWISP.
---

# AGENTS.md

## Project Overview

`ansible-openwisp2` is the official Ansible role for installing and upgrading OpenWISP.

Core code lives in this repository root:

- `tasks/`, `handlers/`, `defaults/`, `vars/`, and `meta/` define role behavior and variables.
- `templates/` provides generated Django, nginx, uWSGI, supervisor, and FreeRADIUS configuration; `files/` contains Python dependency requirement files and the Django secret-key generator.
- `molecule/` contains integration scenarios.
- Documentation lives in `docs/`.
- `docs/` is incorporated into the unified, versioned OpenWISP documentation built by `openwisp-docs`, not a standalone site; use `docs/user/` for end users and `docs/developer/` for contributors and developers of extensions, downstream, or derivative apps.

## Source of Truth

- Use `README.md` and `docs/` for setup, role variables, and deployment behavior.
- Use `.github/workflows/ci.yml`, `.ansible-lint`, and `.yamllint.yml` for CI-tested QA and test commands.
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
- Preserve role variables, defaults, handlers, task ordering, idempotency, supported OS behavior, and upgrade paths unless explicitly required.
- Be careful with templates, secrets, file ownership, permissions, service restarts, migrations, and FreeRADIUS/nginx/supervisor integration.
- Be careful when changing nginx listen directives, FreeRADIUS listen addresses, or service ports. Do not broaden a service's network exposure beyond the role's documented deployment configuration without an explicit requirement.
- Process inventories, API responses, generated work, and telemetry in bounded batches.
- Do not accumulate all pages, task results, logs, or queued telemetry in memory without a known bound.
- Keep buffers and retry queues bounded. Define what happens when the limit is reached, such as sending the current batch, dropping old data, or reporting an error.
- When code consumes a paginated API, follow its continuation mechanism and process one page at a time.
- Avoid unnecessary blank lines inside Jinja, YAML, and shell blocks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwisp/ansible-openwisp2](https://github.com/openwisp/ansible-openwisp2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
