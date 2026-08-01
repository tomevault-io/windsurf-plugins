---
trigger: always_on
description: <!-- SPECKIT START -->
---

# Toga - Agent Development Guide

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure, shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

This file tells autonomous coding agents (and human contributors using agent tools) how to be productive in this repository without breaking it. It is operational guidance; the binding rules live in `.specify/memory/constitution.md` and the contribution docs under `docs/en/how-to/contribute/`. When this file disagrees with either, the constitution wins.

## What Toga is

Toga is a Python-native, OS-native GUI toolkit published under BSD-3-Clause as part of the BeeWare suite. A single `toga-core` API is rendered by a platform-specific backend so that one Python codebase produces native apps on macOS, Windows, Linux, iOS, Android, Web, and terminal.

## Repository layout

| Path | What it contains |
| --- | --- |
| `core/` | `toga-core` — the public API and shared widget contracts. |
| `travertino/` | Style and layout engine used by `core`. |
| `dummy/` | Reference headless backend; MUST implement the full core API so core tests can run without a GUI. |
| `cocoa/` | macOS backend. |
| `gtk/` | Linux/GTK backend (GTK3 stable, GTK4 experimental). |
| `winforms/` | Windows backend. |
| `iOS/` | iOS backend. |
| `android/` | Android backend. |
| `textual/` | Terminal backend. |
| `web/` | Web backend. |
| `qt/` | Linux/Qt backend. |
| `positron/` | Briefcase plugin for generating apps where the UI has been defined with web tools. Similar to Electron, but using Python for the web server |
| `testbed/` | Briefcase app used to validate backend behaviour on a real platform. |
| `docs/en/` | User and contributor documentation (MkDocs, `mkdocs.en.yml`). |
| `changes/` | Towncrier fragments (`<issue>.<kind>.md`) — one per user-visible change. |
| `examples/`, `demo/` | Standalone sample apps. |
| `.specify/` | SpecKit workflow assets (constitution, templates, extensions). |

Do not create new top-level directories without a clear constitutional reason; extend existing ones.

## Non-negotiables (from the constitution)

1. **Backend parity.** A new `toga-core` API MUST have a concrete implementation plan for every production backend. The Dummy backend MUST always implement the full surface.
2. **Native behaviour.** Production backends MUST delegate to the native toolkit. No emulated/custom-drawn widgets without documented justification.
3. **Comprehensive tests (NON-NEGOTIABLE).** 100% line coverage for `core` and `travertino`. Backend changes MUST be exercised via `testbed/`. Bug fixes MUST include a regression test that fails before the fix.
4. **Documented change.** Every user-visible change MUST include a `changes/<issue>.<kind>.md` fragment and updated docs. Breaking changes require a deprecation release.
5. **Contributor accountability.** The submitter owns the diff, regardless of AI assistance. Follow the BeeWare Code of Conduct and AI Policy.

Any PR failing one of these is rejected, not waived.

## Toolchain

- **Python**: 3.10–3.14 (see `core/pyproject.toml` classifiers).
- **Task runner**: `tox` (with `tox-uv`). Install the dev tooling via `uv pip install --group dev` at the repo root, or let `tox` bootstrap.
- **Lint/format**: `ruff` (check + format), `codespell`, `rumdl` (Markdown), configured in root `pyproject.toml`.
- **Pre-commit**: `pre-commit run --all-files` — MUST pass before PR.
- **Packaging / testbed driver**: `briefcase`.
- **Release notes**: `towncrier` (config in root `pyproject.toml`).
- **Docs**: MkDocs; built with the `docs` dependency group.

Do not replace or bypass these tools. Add new dependencies only with a clear need and a compatible license (BSD-3-Clause friendly).

## Canonical commands

Run from the repository root unless noted.

```console
# Everything pre-commit checks (ruff, format, codespell, rumdl, etc.)
pre-commit run --all-files

# Core + Travertino test suites with coverage (MUST be 100%)
tox -m test

# Just core
tox -m test-core

# Just Travertino
tox -m test-trav

# A single test file against core
tox -e py-cov -- core/tests/path/to/test_file.py

# A single test file against Travertino
tox -e py-trav -- travertino/tests/path/to/test_file.py

# Towncrier draft (preview assembled release notes)
tox -e towncrier-check

# Docs lint
tox -e docs-lint
```

### Testbed (backend validation)

The core suite uses the Dummy backend. Real backend behaviour is validated through the testbed app. Install only the backend under test in your virtualenv, then:

```console
# Desktop (from testbed/)
briefcase dev --app testbed --test

# GTK variants (Linux)
briefcase dev --app testbed --test
TOGA_GTK=4 TOGA_GTKLIB=None briefcase dev --app testbed --test
TOGA_GTK=4 TOGA_GTKLIB=Adw briefcase dev --app testbed --test

# Qt (Linux)
briefcase dev --app testbed-qt --test

# Textual (Linux)
briefcase dev --app testbed-textual --test

# Mobile (requires the relevant SDK/toolchain)
briefcase run android --app testbed --test
briefcase run iOS --app testbed --test   # macOS host only

# Subset with slow mode for visual inspection
briefcase dev --app testbed --test -- tests/widgets/test_button.py --slow
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beeware/toga](https://github.com/beeware/toga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
