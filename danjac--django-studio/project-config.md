---
trigger: always_on
description: This is a Copier template to create a new Django project.
---

This is a Copier template to create a new Django project.

## General Rules

STOP and ask before applying a fix. Before making any changes, state your hypothesis about
the root cause and which files you plan to modify. Wait for confirmation before proceeding.
If the user corrects you, re-read the relevant code/docs before responding.

Do not make excessive or environment-specific changes beyond what was requested. If the
user asks for a targeted edit, make only that edit. Ask before adding dev/staging/production
variants.

When the user corrects you or says 'stop', immediately stop, acknowledge the correction,
and re-examine the situation from scratch. Do not repeat the same approach or question the
user's stated facts.

## Testing

Create a new project in `/tmp` using the Copier template:

```bash
uvx copier copy --trust --defaults --data project_name="My App" . /tmp/my_app
```

If a project has already been created in `/tmp`, remove it first (stop services to avoid port conflicts):

```bash
cd /tmp/my_app
just stop
cd ..
trash my_app
```

Then navigate to the generated project and test it:

```bash
cd /tmp/my_app
just start                      # start Docker services
just install                    # copies .env.example→.env, git init, Python deps, pre-commit hooks
just lint                       # run linters
just typecheck                  # run type checks
just dj makemigrations          # generate initial users migration (expected on first run)
just test                       # run tests
just test-e2e                   # run Playwright E2E tests
just stop                       # stop Docker services
```

**Pre-commit must report no lint errors** on the generated project. Auto-formatters
(ruff-format, DjHTML, DjCSS, Djade) modifying files is expected and fine. Actual
lint errors (`ruff check`, `djlint --lint`) must be zero. Run after install:

```bash
cd /tmp/my_app && uv run pre-commit run --all-files
```

**Expected on first run:**

- `just test` will report coverage below 100%. This is normal - the template ships utility
  modules (`admin.py`, `db/search.py`, `http/`) as starting points without tests. The 100%
  coverage target is a goal for developers to reach as they build their project, not a
  gate on the initial scaffold.
- `just dj makemigrations` is required before `just test` because the `users` app ships
  without an initial migration. Developers are expected to customise the `User` model
  fields before generating migrations.

**Note:** Run `uv sync` from the repo root first if working from a fresh clone and the virtual environment doesn't exist yet. This installs the Copier tooling itself; it is separate from the generated project's dependencies.

## Template Project Testing

The template project itself has its own tests and pre-commit config. Run these from the root directory:

```bash
just check      # Run lint, format, and tests
just test       # Run pytest
just lint       # Run ruff on tests
just format     # Check formatting
```

To install pre-commit hooks for the cookiecutter project itself:

```bash
uv run pre-commit install
```

Run pre-commit hooks on all files in the cookiecutter project to ensure all checks pass:

```bash
git init && git commit -A && pre-commit run --all-files
```

## Working on the Template

Before adding or modifying any code under `template/`, read the generated project's documentation first:

- `template/AGENTS.md.jinja` — agent workflow, conventions, and command reference
- `template/docs/` — all reference docs (Django config, views, models, packages, HTMX, etc.)

These docs describe the patterns, types, and conventions the generated project uses. Code that ignores them will be wrong.

**Naming convention:** all files under `template/docs/` and `template/.agents/skills/*/references/` use lower-kebab-case (e.g. `django-views.md`, `help.md`). New docs and reference files must follow the same pattern.

After any change to `template/`, regenerate the project and verify all checks pass before committing:

```bash
cd /tmp && trash my_app && find /tmp/.Trash-1000 -mindepth 1 -delete
uvx copier copy --trust --defaults --data project_name="My App" . /tmp/my_app
cd /tmp/my_app
git init && git add -A
uv run pre-commit run --all-files   # run twice if hooks auto-fix files
uv run pre-commit run --all-files
just typecheck
```

**Always trash and delete the old project before regenerating.** Stale generated
projects cause `just check` to pass locally while CI fails — the repo-level tests in
`tests/test_project.py` regenerate from scratch every run and catch mismatches that
a cached `/tmp/my_app` hides.

**Do not commit until all checks are clean.**

**Jinja2 processing:** Only files with a `.jinja` suffix are processed by Copier's Jinja2 engine; all other files are copied verbatim. Files that contain conflicting `{{ }}` syntax (e.g. `justfile` uses `{{ args }}`, GitHub Actions workflows use `${{ }}`) must remain plain files and use `PROJECT_SLUG` as a plain-text placeholder, substituted by the hook.

## UI Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danjac/django-studio](https://github.com/danjac/django-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
