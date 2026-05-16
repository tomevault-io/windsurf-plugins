---
trigger: always_on
description: CRITICAL: When starting new work, make sure to check the wiki for relevant documentation and update accordingly.
---

# CLAUDE.md

CRITICAL: When starting new work, make sure to check the wiki for relevant documentation and update accordingly.
As you make progress, make sure to periodically update the wiki so that nothing is inconsistent with the code base.

## Stack at a glance

- **Backend** — FastAPI (uvicorn) + Postgres 17 (with `pg_textsearch` for BM25 search and `pgmq` for the task queue) + custom workers. Git is shelled out to.
- **Frontend** — Next.js 14 (App Router) + TypeScript.
- **Nginx** in front, reverse-proxying `/api/*` → backend, everything else → frontend.
- App state and queues both live in Postgres (connection via `DATABASE_URL`). Wiki working tree on volume `wiki-data`.

See `docs/architecture.md` for the data flows.

## Pre-commit hooks

**Run `pre-commit install` once after cloning** — every commit then runs the
same checks CI runs on PRs, so type or lint failures surface locally before
they hit a review. Hook config lives in `.pre-commit-config.yaml`. Run
`pre-commit run --all-files` ad-hoc to lint the whole tree.

What runs on backend Python files:

- **ruff** (`backend/pyproject.toml [tool.ruff]`) — fast lint.
- **basedpyright** (`backend/pyproject.toml [tool.basedpyright]`, strict mode)
  — type check. Runs whole-project on any backend `.py` change because it
  needs the import graph; passing only the changed files would miss
  cross-file errors.

Both hooks resolve their version via `uv run --project backend --extra dev`
so the pre-commit run uses the exact same tools as `uv sync --extra dev`.
Make sure you've run `uv sync --extra dev` in `backend/` first.

Add new checks as hooks here, not as one-off CI steps.

## Layout

```
backend/app/
  api/            FastAPI routers — thin HTTP layer, no business logic
  auth/           sessions, bcrypt, whitelist, admin flags
  db/             SQLAlchemy ORM models (Postgres + pg_textsearch BM25)
  llm/            provider-agnostic LLM client + DB-backed settings + agents
  models/         pydantic schemas (request/response shapes)
  tasks/          tasks (workers run in their own container)
  triggers/       NL-trigger evaluation engine
  wiki/           git subprocess wrapper + path utilities + search
frontend/src/
  app/            Next.js routes
  components/     UI components (AppShell, etc.)
  lib/            api.ts, auth.tsx — the only place pages talk to the network/auth
  types/          shared TS types
nginx/            reverse proxy
wiki/seed/        sample content for fresh installs
docs/             architecture + API reference
```

## Architectural rules — required interfaces and seams

These exist so the system stays testable and swappable. Honor them.

### LLM calls — always through `app/llm/client.py`

`stream(messages, ...)` and `complete(messages, ...)` (a drainer) in
`app/llm/client.py` are the **only** allowed entry points for talking to a
model. They yield/return a normalized shape (`text_delta`/`tool_call`/`done`
events; `{text, tool_calls, stop_reason, usage}` dicts) so callers don't
branch on provider.

Provider implementations live as a plural seam under `app/llm/providers/`:
one module per backend (`anthropic.py`, `openai.py`, `gemini.py`, `ollama.py`),
each exposing a module-level `PROVIDER` satisfying the `Provider` protocol
(`name`, `check_configured(settings)`, `stream(messages, *, model, tools,
max_tokens, settings)`).

- Do **not** `import anthropic`, `import openai`, `from google import genai`,
  or `import ollama` outside the matching `app/llm/providers/<name>.py` module.
- Provider, model, and credentials come from `app/llm/settings.py:get()`
  (DB-backed, configured via the admin page). Don't read provider keys from
  `CONFIG` or `os.environ` anywhere else.
- Add a new provider by dropping `app/llm/providers/<name>.py` with a
  `PROVIDER` instance and importing+registering it from
  `app/llm/providers/__init__.py`. Don't add if/elif branches in `client.py`.
- In tests, patch `app.llm.client.stream`/`complete` for caller-level tests,
  or the per-provider `_client` for SDK-shape tests. Never import the real
  provider SDKs in tests.

### Auth — `Depends(...)`, not raw session reads

In API code, gate routes with `Depends(require_user)` or
`Depends(require_admin)` from `app.auth.deps`. The dependency
function returns a typed `User`; bind it on the route signature and
pass it down (e.g. to `require_can(action, path, user)`). For
non-HTTP code paths (worker tasks, agent tools), read the active
user with `app.auth.current_user()` — it reads the ContextVar bound
by ``app.auth.deps.CurrentUserMiddleware`` for HTTP requests or by
``set_current_user(user)`` for background tasks. Don't touch
`request.session["user_id"]` outside `app/api/auth.py`.

- Public endpoints (signup, login, `/auth/config`, inbound webhooks)
  are explicit — everything else takes a `Depends(require_user)`
  parameter.
- The first registered user is auto-admin (`users_repo.create` checks
  `count() == 0`). Admin can't be left at zero (see `app/api/admin.py`
  — demote/delete guard against `admin_count() <= 1`).
- The session cookie is signed by Starlette's ``SessionMiddleware``
  (installed in ``app/main.py:create_app``); ``app.auth.deps.current_user``
  reads ``request.session["user_id"]``.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onyx-dot-app/agent-wiki](https://github.com/onyx-dot-app/agent-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
