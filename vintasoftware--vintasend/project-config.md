---
trigger: always_on
description: VintaSend is a Python library for transactional notifications. It records every notification in a
---

# VintaSend

VintaSend is a Python library for transactional notifications. It records every notification in a
data store, renders it from a template at send time, and dispatches it through a pluggable adapter
(email, SMS, push, in-app). The library itself is deliberately incomplete: it defines three abstract
seams and ships fakes for them, but every real backend, adapter, and template renderer lives in a
separate `vintasend-*` package. This repository owns the interfaces, the orchestration, and the
contract — not the integrations.

It is published to PyPI as `vintasend` and consumed by external Python applications, so the public
API and its type signatures are the product.

## Project Overview

Single package, no monorepo tooling.

- **`vintasend/services/notification_service.py`** — the entry point. `NotificationService` and
  `AsyncIONotificationService`, plus the `@register_context` decorator and the `Contexts` registry.
- **`vintasend/services/notification_backends/`** — the storage seam. Where notifications are
  persisted and queried.
- **`vintasend/services/notification_adapters/`** — the delivery seam. How a notification actually
  gets sent.
- **`vintasend/services/notification_template_renderers/`** — the rendering seam. How a notification
  body is produced from a template plus context.
- **`vintasend/tasks/`** — `background_tasks.py` and `periodic_tasks.py`, the hooks a host app wires
  into Celery, cron, or similar to drain pending notifications.
- **`vintasend/app_settings.py`** — settings resolution across Django, Flask, FastAPI, and bare env
  vars.
- **`vintasend/tests/`** — the whole suite, run against the in-repo fakes.

**There is no database, no web server, and no frontend in this repo.** Persistence is entirely the
backend implementation's problem. The test suite runs against `FakeFileBackend`, which serializes to
a JSON file. Nothing here opens a socket except `requests`, used to fetch URL-sourced attachments.

## Tech Stack

- **Python 3.10–3.14** — the floor is 3.10 and it is load-bearing. See **Supporting the minimum
  Python** below.
- **Poetry** — dependency management and publishing.
- **Runtime dependencies are intentionally tiny**: `typing-extensions`, `packaging`, `requests`.
  Adding a fourth is a real decision, not a routine one.
- **pytest** (`^9`) with `pytest-xdist` (`--dist=loadscope`), `pytest-asyncio`, `pytest-cov`, and
  `freezegun` for time control.
- **tox** — runs the suite against all five supported interpreters.
- **ruff** — lint and format, single source of truth for style.
- **mypy** — type checking. The package ships `py.typed`, so its annotations are part of the public
  contract.
- **pre-commit** — ruff, mypy, and hygiene hooks.
- **GitHub Actions** — `ci.yml` (lint + type-check job, then the 3.10–3.14 test matrix),
  `publish.yml` (tag-triggered PyPI release).

## Common Commands

```bash
poetry install --with dev          # install, including dev tooling

poetry run pytest                  # full suite (fast — no I/O, no DB)
poetry run pytest vintasend/tests/test_services/test_notification_service.py   # scoped
poetry run tox                     # the suite across Python 3.10–3.14

poetry run ruff check .            # lint
poetry run ruff format .           # format
poetry run mypy                    # type-check (config selects the package)

poetry run pre-commit run --all-files

poetry build                       # build sdist + wheel
```

## Code Style

Enforced by ruff — do not hand-format, run `poetry run ruff format .`:

- Line length 100, 4-space indent, double quotes.
- Two blank lines after the import block (`lines-after-imports = 2`).
- isort ordering with a dedicated `django` section between stdlib and third-party.

Conventions ruff does not enforce:

- **Type everything.** The package ships `py.typed`. A new public method without annotations is
  incomplete.
- **Guard type-only imports.** Anything imported purely for annotations goes inside
  `if TYPE_CHECKING:` with string annotations at the use site. This keeps the import graph flat and
  avoids cycles between the service and the seams.
- **Import `NotificationContextDict` and the other dataclasses from
  `vintasend.services.dataclasses`**, which is where they are defined — not from
  `notification_service`, which merely re-exports them. `no_implicit_reexport` is on and will reject
  the indirect path.
- **Exceptions derive from `NotificationError`** in `vintasend/exceptions.py`, which itself derives
  from `ValueError`. Raise the most specific existing subclass; add a new one rather than raising a
  bare `ValueError`.
- **Singletons use `SingletonMeta`** from `vintasend/utils/singleton_utils.py`. `NotificationSettings`
  and `Contexts` are the two instances. Do not invent a second singleton mechanism.

### Supporting the minimum Python

The floor is Python 3.10 and several settings encode it. Do not raise any of them to match whatever
interpreter you happen to be running:

- `[tool.ruff] target-version = "py310"` — set higher, pyupgrade rewrites code into 3.11/3.12-only
  syntax that breaks the tox matrix.
- `[tool.mypy] python_version = "3.10"` — set higher, errors that only appear on the oldest
  interpreter get hidden.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vintasoftware/vintasend](https://github.com/vintasoftware/vintasend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
