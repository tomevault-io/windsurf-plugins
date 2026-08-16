---
trigger: always_on
description: `django-rest-framework-gis` adds GIS fields, serializers, filters, pagination, and schema helpers to Django REST Framework.
---

# AGENTS.md

## Project Overview

`django-rest-framework-gis` adds GIS fields, serializers, filters, pagination, and schema helpers to Django REST Framework.

Core code lives in `rest_framework_gis/`:

- `fields.py`, `serializers.py`, `filters.py`, `filterset.py`, `pagination.py`, `schema.py`, and `tilenames.py` implement GIS API helpers.
- Tests live in `tests/`.

## Source of Truth

- Use `README.rst`, `CHANGES.rst`, and `performance_tests.rst` for setup, package usage, and performance context.
- Use `.github/workflows/ci.yml` and `tox.ini` for CI-tested dependencies, QA/test commands, env vars, and supported Python/Django/DRF versions.
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
- Preserve public APIs, serializer output formats, GeoJSON compatibility, filter semantics, and pagination behavior unless explicitly required.
- Mark user-facing strings for translation with Django i18n helpers in Django code.
- Place imports at the top of the file. Only defer imports when necessary (e.g., Django model imports inside functions or methods where the app registry is not yet ready).
- Avoid unnecessary blank lines inside function and method bodies.
- Prefer short, precise names that rely on their nearest meaningful scope. Do not repeat a feature, domain object, or namespace already named by the containing module, class, or function. For example, prefer `EstimatedLocation.refresh()` over `EstimatedLocation.refresh_estimated_location()`. Repeat that context only when the name is used outside that scope or is needed to distinguish genuinely different concepts. When a concise name cannot express a necessary distinction, use a concise docstring to describe it rather than encoding it in an excessively long name.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwisp/django-rest-framework-gis](https://github.com/openwisp/django-rest-framework-gis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
