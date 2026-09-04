---
trigger: always_on
description: This document provides **implementation and contribution guidelines** for developing new features, modules, or commands within the `python-moodle` project.
---

# AGENTS.md

## Purpose

This document provides **implementation and contribution guidelines** for developing new features, modules, or commands within the `python-moodle` project.
It is designed for both human developers and AI agents that will generate, review, or maintain code in this repository.

---

## General Philosophy

* **Write everything in English:**
  All code, docstrings, function names, variable names, and code comments **MUST** be written in English, regardless of the language of user requests or documentation.
* **Pythonic by design:**
  Code must follow the [PEP8](https://peps.python.org/pep-0008/) style guide. Prioritize clarity, modularity, and explicitness.
* **CLI-first and library-ready:**
  The CLI (`cli.py`) should be a thin layer over importable Python modules. Every CLI command should map directly to a public function in the corresponding module.
* **Testability:**
  All features must be accompanied by simple, readable unit tests (in `tests/`). Use real Moodle sandboxes when needed, but support mocking.
* **Extensibility:**
  Structure code so that new module types (e.g., quiz, assign) can be added with minimal coupling.

---

## Project Structure

```
python-moodle/
├── src/
│   └── py_moodle/
│       ├── __init__.py         # Re-exports MoodleClient, models, MoodleSession, Settings
│       ├── __main__.py
│       ├── py.typed            # PEP 561 marker (the package ships type information)
│       ├── cli/                # Typer CLI layer (thin wrappers over the library)
│       │   ├── app.py          # Root Typer app + global flags (--output/--quiet/--verbose/...)
│       │   ├── output.py       # OutputFormat, emit(), --fields, console + logging helpers
│       │   ├── courses.py, sections.py, modules.py, users.py, categories.py,
│       │   ├── folders.py, pages.py, resources.py, urls.py, site.py, doctor.py, admin.py
│       ├── client.py           # MoodleClient facade (courses/sections/modules/... namespaces)
│       ├── session.py          # MoodleSession: cached auth + token/sesskey
│       ├── auth.py             # login / CAS SSO (redacted --debug tracing)
│       ├── http.py             # Centralized requests: timeouts, bounded GET retry, redaction, tracing
│       ├── transport/          # webservice.py / ajax.py / html.py strategies
│       ├── compat.py           # Version-sensitive HTML parsing strategies (4.x/5.x)
│       ├── config.py           # Shared HTTP timeout policy
│       ├── models.py           # Typed dataclasses: Course, CourseSection, CourseModule, User, ...
│       ├── ensure.py           # Idempotent ensure_module/label/resource/folder/section
│       ├── doctor.py           # Environment self-diagnostics
│       ├── course.py, section.py, label.py, resource.py, folder.py, page.py,
│       ├── url.py, module.py, user.py, category.py, scorm.py, draftfile.py,
│       ├── upload.py, permissions.py, settings.py, site.py, assign.py
├── tests/
│   ├── unit/                   # No Moodle needed; run in CI on every push (make test-unit)
│   │   ├── fixtures/html/       # Captured HTML for compat-parser regression tests
│   │   └── test_*.py
│   ├── conftest.py             # Shared fixtures: create_temporary_course, course_creation_lock, login warmup
│   ├── test_course.py, ...     # Integration tests (need --integration + a live Moodle)
├── docs/                       # MkDocs/Zensical site (recipes.md, api/, roadmap-plan.md, ...)
├── README.md, LICENSE, pyproject.toml, .env.example, docker-compose.yml, Makefile
```

---

## Architecture (current layering)

New code MUST fit this layering — do not call `requests`/`session.get` directly from
feature modules, and do not put Moodle logic in the CLI.

1. **CLI layer (`cli/`)** — Typer sub-apps, one file per entity, each command a thin
   wrapper that calls a public library function and renders via `cli/output.py`.
   * Machine-readable output is unified: every `list` command supports
     `--output table|json|yaml|csv` and `--fields field1,field2` (order-preserving
     projection for json/yaml/csv). Use `emit(data, output_format, table_fn=..., csv_fields=..., fields=...)`.
   * Global flags on the root app: `--quiet`, `--no-color` (also `NO_COLOR`),
     `--verbose`/`-v` (INFO), `--debug` (DEBUG). Diagnostics go to **stderr** only, so
     they never contaminate `--output json`/`csv` on stdout.
2. **Library layer** — one module per Moodle entity (`course.py`, `section.py`, `label.py`,
   `resource.py`, `folder.py`, `page.py`, `url.py`, `module.py`, `user.py`, `category.py`,
   `scorm.py`, `draftfile.py`, `upload.py`). Every public function is import-friendly and
   CLI-agnostic, takes an explicit `session`/`base_url`/`sesskey`, and raises a typed
   `Moodle<Entity>Error`.
3. **HTTP / transport layer** — all new HTTP goes through `http.py`
   (`request_webservice`/`request_html_get`/`request_form_post`/`request_ajax`/`upload_file`):
   it applies the shared timeout policy from `config.py`, retries **only** idempotent
   GET-style requests with bounded backoff (mutations are never auto-retried), redacts
   secrets from any exception message, and emits redacted `DEBUG` request traces on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erseco/python-moodle](https://github.com/erseco/python-moodle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
