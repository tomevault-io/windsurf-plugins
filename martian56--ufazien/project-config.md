---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

Ufazien is a student platform: grade calculators, a blog, a community with real-time chat, a mini web-hosting service for student projects, and a multiplayer 3D campus with proximity voice. Django REST API plus a React SPA. It is deployed and has real users, so changes here reach people.

## Layout

```
backend/     Django project `ufazien`, one app per feature
frontend/    React 19 + Vite SPA
hosting/     nginx + php-fpm compose resource serving *.ufazien.com user sites
```

Backend apps: `api` `users` `blog` `gpa` `average` `game` `ai_tools` `hosting` `community` `schedule`.

Two naming traps:

- **`game`** is the campus simulator. **`schedule`** is the calendar. It is not called `calendar` because a top-level `calendar` package would shadow the standard library module Django itself imports.
- **`hosting`** is the mini-PaaS (user websites and databases). The `hosting/` directory at the repo root is unrelated: it is the nginx/php compose stack.

## Running it

```bash
# backend
cd backend && python -m venv .venv && source .venv/Scripts/activate
pip install -r requirements-dev.txt
SECRET_KEY=dev python manage.py migrate
SECRET_KEY=dev python manage.py runserver

# frontend
cd frontend && bun install
VITE_API_URL=http://localhost:8000 bun run dev
```

`SECRET_KEY` is required. The database is SQLite unless `ENVIRONMENT=production`, so no server is needed locally.

Use `requirements-dev.txt`, not `requirements.txt`: the WebSocket tests import `channels.testing`, which pulls in `daphne`. Production serves ASGI with uvicorn and does not install it.

## Testing

```bash
cd backend && SECRET_KEY=test python manage.py test          # all
cd backend && SECRET_KEY=test python manage.py test community
```

Add tests for what you change. Most apps' `tests.py` began as an empty stub, and everything they now cover was a bug that reached production unnoticed.

The frontend runs Vitest: `bun run test`, with `bun run typecheck` for types. Coverage is thin and
starts at the API client, so a browser check still matters for anything visual. Say what you checked.

## Rules that matter here

**A user's email must never reach another user.** `email` is a `SerializerMethodField` that returns the address only to its owner, and `None` otherwise, including when there is no request in context, which is how WebSocket consumers serialize. See `community/serializers.py`; `community/tests.py` guards it. This leaked in production once.

**Scope user-owned data to its owner in `get_queryset`.** Another user's record should 404, not merely be absent from a list. `schedule/views.py` is the reference.

**Permissions are decided server-side.** The LiveKit token lists exactly which sources a participant may publish, derived from `LobbyMember` fields, so a modified client cannot grant itself the microphone or a screen share. Never move that decision into the browser.

**Never commit secrets.** `.env`, keys and certificates are gitignored. A private key was committed here once and is still in history.

**`SECRET_KEY` is read from the environment and has no fallback.** There used
to be a literal default, which made the published contents of this repository
the signing key for anybody who had not set the variable — and everything
Django signs comes from it, including the JWTs the API authenticates with.
Guarding it behind `DEBUG` was not enough, because a box brought up with
`DJANGO_DEBUG=true` still ran on the published key, so the default is gone and
Django refuses to start without one in any mode. `SECRET_KEY=dev` locally, as
the commands above already show. Setting it in production for the first time
signs everybody out once; that is the rotation working.

**Credentials are minted server-side, and a password is rotated on the server
that holds it.** A hosting database's `username` and `password` are read-only
on the serializer: the browser used to generate both with `Math.random()` and
post them, and what it sent became the real credential. `change_password` runs
`ALTER ROLE`/`ALTER USER` through `set_database_password` and writes the row
only once the server has accepted it — it used to assign the field and save, so
the dashboard showed a new password while the real user kept the old one.

**A site's files are reached through `path_within`, never `startswith`.**
`/srv/hosting/alice` starts with `/srv/hosting/a`, and people choose their own
subdomains, so the prefix check that used to guard `delete_file` and
`download_file` let a site called `a` read and delete files in every site whose
name began with an `a`. It resolves with `realpath` and compares with
`commonpath`, which also closes a symlink planted inside the site.

**Subdomains go through `hosting/domains.py`.** The name becomes a directory on
disk and the root nginx serves, so it is not free text: `check()` lower-cases
it, rejects anything that is not a hostname, and refuses the reserved list —
`admin`, `login`, `api` and the rest. A site on `login.ufazien.com`, served
under the platform's own wildcard certificate, is a convincing place to ask
somebody for a password.

**Every site shares one php-fpm pool, so `open_basedir` is what separates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martian56/ufazien](https://github.com/martian56/ufazien) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
