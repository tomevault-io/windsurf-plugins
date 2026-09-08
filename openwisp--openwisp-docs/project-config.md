---
trigger: always_on
description: `openwisp-docs` builds the unified OpenWISP documentation published on openwisp.io.
---

# AGENTS.md

## Project Overview

`openwisp-docs` builds the unified OpenWISP documentation published on openwisp.io.

Core content lives in this repository root:

- `user/`, `developer/`, `general/`, `tutorials/`, and `releases/`, plus top-level `.rst` files such as `modules.rst`, contain local documentation content.
- `build.py` fetches module documentation listed in `config.yml` into generated `staging-dir/`, combining it with local content into versioned HTML, PDF, and ePub output and curated cross-project release notes.
- Fetched module `docs/` directories are inputs to this unified documentation build, not standalone documentation sites.
- Documentation separates end-user material in `user/` from developer material in `developer/`, which serves core contributors and developers building extensions, downstream apps, or other derivative work. End-user pages may link to developer documentation for advanced topics.
- `_build/` and `staging-dir/` contain generated build output.
- `_static/`, `_templates/`, `_styles/`, `assets/`, `images/`, and `partials/` contain presentation assets and shared snippets.
- `conf.py`, `config.yml`, `build.py`, `Makefile`, and helper scripts support documentation builds and releases.

## Source of Truth

- Use `README.rst` and `developer/contributing.rst` for setup, style, and contribution guidance.
- Use `.github/workflows/ci.yml`, `pyproject.toml`, and `requirements.txt` for CI-tested docs build, QA, spelling, and link checks.
- Use GitHub issue/PR templates when asked to open issues or PRs.

If instructions conflict, repository config and CI workflows win first, docs next, and this file is supplemental.

## Contributing Guidelines

- Before editing, inspect the relevant implementation, tests, documentation, and configuration. Follow existing repository patterns and do not invent behavior or requirements.
- Keep each contribution focused and change only the lines necessary for its goal. Do not include unrelated refactors, formatting churn, or generated and dependency-file changes unless explicitly required.
- Run `openwisp-qa-format` after each change when available.
- Run the relevant targeted tests, builds, and documented QA checks, including `./run-qa-checks` when provided. Do not claim a change is complete when verification fails; report the failure or blocker.
- When requirements, intended behavior, or an unexpected failure are unclear, stop and seek clarification instead of making speculative changes.
- When starting work on a new issue, create a new branch from `master`. Use `issues/<issue-number>-<short-title>` for issue work; otherwise, use a short, descriptive branch name.
- Commit messages must be descriptive and use past tense. Past tense is a writing guideline that agents and contributors must follow; it is not checked automatically. For issue work, use an allowed prefix and a capitalized, past-tense subject ending with `#<issue-number>`, for example `[fix] Fixed perennial "modified" state #213`. Repeat the issue reference in the body with `Fixes`, `Closes`, `Resolves`, or `Related to` as appropriate. After creating a commit, use `openwisp-commit --check` to validate the current `HEAD`; it cannot validate a proposed message. Use `openwisp-commit --check --rev-range <range>` for an existing commit range, and `cz -n cz_openwisp info` to view allowed prefixes and message structure.
- Add an explanatory commit body only for substantial changes, new features, or non-obvious bug fixes. The releaser automatically publishes the subject of `[feature]`, `[change]`, `[change!]`, `[deps]`, and `[fix]` commits, including scoped variants, in the changelog. Write those subjects in clear, user-friendly language suitable for release notes.
- Send new commits in response to review feedback instead of amending existing commits.

## Development Rules

- Follow the DRY principle: do not duplicate information or code across files.
- Preserve headings, anchors, cross-references, versioned links, include directives, image paths, and public URLs unless explicitly required.
- Edit local content or source module documentation, then rebuild instead of editing `_build/` or `staging-dir/` directly.
- Be careful with release notes, version switcher behavior, generated files, spell-check word lists, and module documentation copied from upstream repos.
- Avoid unnecessary blank lines inside directive blocks, literal blocks, and helper functions.
- Prefer short, precise names that rely on their nearest meaningful scope. Do not repeat a feature, domain object, or namespace already named by the containing module, class, or function. For example, prefer `EstimatedLocation.refresh()` over `EstimatedLocation.refresh_estimated_location()`. Repeat that context only when the name is used outside that scope or is needed to distinguish genuinely different concepts. When a concise name cannot express a necessary distinction, use a concise docstring to describe it rather than encoding it in an excessively long name.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwisp/openwisp-docs](https://github.com/openwisp/openwisp-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
