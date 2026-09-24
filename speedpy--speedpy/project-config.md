---
trigger: always_on
description: This file is the source of truth for AI Coding Agents working in this repository.
---

# AGENTS.md

This file is the source of truth for AI Coding Agents working in this repository.
`CLAUDE.md` and `.cursorrules` exist only as pointers to this file — keep guidance here.

## Additional case-specific instructions

Read the matching file **before** you start, if your task is one of these. They
live outside this file so that a capability most projects never touch does not
cost every reader the tokens to scroll past it.

- video manipulation, transcoding, compression, video upload, ffmpeg: `agents_docs/working_with_video_files.md`
- hosted remote MCP, MCP endpoint, MCP connector, OAuth for AI agents, streamable HTTP, mcp.<domain>, RFC 8707 audience, CIMD: `agents_docs/working_with_hosted_mcp.md`

Adding one: keep the file in `agents_docs/`, name it for the task rather than the
technology (`working_with_<thing>.md`), and add exactly one bullet here —
searchable words first, path last. Anything a project would only sometimes need
belongs there, not in this file.

## Project Overview

SpeedPy Standard is a Django-based web application starter template featuring a single-app architecture with custom
user authentication, Celery for background tasks, and Tailwind CSS for styling. The project supports two development
modes — Docker Compose or local uv + npm — picked at init time.

## Development Commands

For mode-specific run commands (Django management, Tailwind, migrations, tests, shell access, etc.) read
**`AGENTS-local.md`** in this directory. It is populated by `init-docker.sh` or `init-local.sh` and reflects the
active setup. Do not assume a particular wrapper (`docker compose run` vs `uv run`) — the cheat sheet is the source
of truth for invocation.

Initialization scripts:

- `bash init-docker.sh` — boots the project with Docker Compose (Postgres, Redis, Celery, nginx media).
- `bash init-local.sh` — runs the project on the host with uv + npm, SQLite, no Redis, Celery in always-eager mode.

## Architecture

### Apps Structure (single-app architecture)

This is a **single-app Django project**. All business logic ships from `mainapp`.
**Do not create a new Django app** for a new feature, page, or model — extend `mainapp`.
The only existing apps and their narrow purposes:

- **`mainapp`** — every piece of business logic, every page, every URL that is not user
  auth and not the boilerplate demo. New models, views, forms, admin, Celery tasks,
  templates all go here. Uses **package-style modules** (one file per concern, re-exported
  through `__init__.py`):
    - `mainapp/models/<group>.py` — re-export the class in `mainapp/models/__init__.py`
      and add it to `__all__`
    - `mainapp/views/<group>.py` — re-export in `mainapp/views/__init__.py` `__all__`
      (class-based views only)
    - `mainapp/forms/<group>.py` — re-export in `mainapp/forms/__init__.py` `__all__`
    - `mainapp/admin/<group>.py` — re-export in `mainapp/admin/__init__.py`
    - `mainapp/tasks/<group>.py` — Celery tasks, re-export in `mainapp/tasks/__init__.py`
    - URL routes in `mainapp/urls.py`
    - Templates in `templates/mainapp/<group>/...`
- **`usermodel`** — custom email-based `User` model and anything that mutates that model
  or its profile (avatar, OTP profile fields owned by the user, allauth adapter,
  signup/login/profile forms). Uses **single-file modules** (`models.py`, `views.py`,
  `forms.py`, `admin.py`, `managers.py`, `adapters.py`) — keep that shape; do not
  package-split it. Anything user-adjacent that is really business logic (e.g. a Team a
  user belongs to, a per-user setting that's part of a feature) belongs in `mainapp`,
  not here.
- **`demoapp`** — read-only reference for boilerplate users. The Product CRUD at
  `/demo/products/` is the canonical example to copy when building new CRUD screens.
  **Do not add real product features here.** If you need to extend the demo to teach a
  new pattern, keep it conventional: `demoapp/models.py`, `demoapp/forms.py`,
  `demoapp/views.py`, `demoapp/urls.py`, templates under `templates/demoapp/`.
- **`speedpycom`** — framework-level utilities shared by every app: `BaseModel` (UUID
  pk + timestamps — inherit it for new models), management commands like
  `generate_tailwind_directories`, default OG image view. Add cross-cutting helpers
  here, not feature code.

DRF (`rest_framework`) and `drf-spectacular` are installed for versioned integration
endpoints under `/api/v1/`. New JSON integration endpoints follow the HTTP API guide
below; do not add DRF to server-rendered HTML views. The primary UI remains
Django templates + crispy forms + Alpine.js. Ad-hoc `JsonResponse`s in
`mainapp/views/` are only for tiny UI helpers (tours, toggles).

### Where new code goes — quick reference

| You're adding…                                | Put it in                                                               |
|-----------------------------------------------|-------------------------------------------------------------------------|
| A business-logic model                        | `mainapp/models/<group>.py` + register in `__init__.py`                 |
| A page / view                                 | `mainapp/views/<group>.py` (class-based) + route in `mainapp/urls.py`   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speedpy/speedpy](https://github.com/speedpy/speedpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
