---
trigger: always_on
description: Coding conventions for this repository. New and changed code follows clean architecture as implemented below.
---

# OpenGateLLM

Coding conventions for this repository. New and changed code follows clean architecture as implemented below.

**Git history is linear.** Never create merge commits (`git merge`, `git pull` without `--rebase`). Update a branch with `git pull --rebase origin main`. After rewriting already-pushed commits, update the remote with `git push --force-with-lease`. Full workflow: [`docs/src/content/docs/contributing/development_environment.mdx`](docs/src/content/docs/contributing/development_environment.mdx).

---

## Architecture

```
domain/           entities, errors, repository ports, query ports + views (no FastAPI, no SQL)
use_cases/        Command, UseCase, UseCaseSuccess, execute()
infrastructure/
  fastapi/        endpoints, schemas, HTTP exceptions, AccessController
  postgres/       Postgres*Repository / Postgres*Query adapters, AutocommitSession
dependencies.py   DI factories (transactional vs autocommit session)
```

**Dependency rule:** `infrastructure → use_cases → domain`.

| Layer | Location |
|-------|----------|
| Domain | `api/domain/<context>/` |
| Use cases | `api/use_cases/<area>/` |
| Endpoints | `api/infrastructure/fastapi/endpoints/` |
| API schemas | `api/infrastructure/fastapi/schemas/` |
| Postgres adapters | `api/infrastructure/postgres/_postgres<entity>repository.py` |
| Read models | `api/domain/<context>/views.py` + `_<noun>query.py` |
| DI | `api/dependencies.py` |

Admin CRUD is grouped per resource: `api/use_cases/admin/roles/`, `api/infrastructure/fastapi/endpoints/admin/roles.py`, `api/infrastructure/fastapi/schemas/admin/roles.py`. Everything else stays flat under its area (`api/use_cases/auth/`, `api/infrastructure/fastapi/endpoints/models.py`, `api/infrastructure/fastapi/schemas/models.py`).

Reference implementations of these patterns:

| Pattern | Example |
|---------|---------|
| Create with domain errors | `api/infrastructure/fastapi/endpoints/admin/keys.py` + `api/use_cases/admin/keys/_createkeyusecase.py` |
| List + get-by-id | `api/infrastructure/fastapi/endpoints/admin/routers.py` (`GetRoutersUseCase` + `GetOneRouterUseCase`), self-service `api/infrastructure/fastapi/endpoints/keys.py` |
| Paginated list | `api/infrastructure/fastapi/endpoints/admin/roles.py`, `api/infrastructure/fastapi/endpoints/admin/users.py` (`EntitiesPage`, `*sResponse`) |
| Full CRUD | `api/infrastructure/fastapi/endpoints/admin/roles.py`, `api/infrastructure/fastapi/endpoints/admin/routers.py` |
| Read-only projection | `api/infrastructure/fastapi/endpoints/models.py` (`ModelQuery` + `ModelView`) |
| Model-forward (autocommit) | embeddings, OCR, rerank, audio transcriptions |

Self-service `/v1/keys` reuses the admin key use cases (`CreateKeyUseCase`, `GetKeysUseCase`, `GetOneKeyUseCase`). Pass `user_id=authenticated_user.id` on the command to scope the operation to the current user. Admin get-one omits `user_id` (it defaults to `None`) so it can load any key.

### `api/endpoints/` — legacy, scheduled for removal

Do not confuse `api/endpoints/` (pre-clean-architecture) with `api/infrastructure/fastapi/endpoints/` (current). **Never add a route or a module to `api/endpoints/`.** Three items remain:

| Remaining | Exposes | Migrate to |
|-----------|---------|------------|
| `api/endpoints/admin/organizations.py` | `PATCH /v1/admin/organizations/{organization}` | `api/infrastructure/fastapi/endpoints/admin/organizations.py` — the other organization routes already live there |
| `api/endpoints/chat.py` | `POST /v1/chat/completions` | `api/infrastructure/fastapi/endpoints/chat.py` + `api/use_cases/chat/` — model-forward, so `AutocommitSession` |
| `api/endpoints/monitoring.py` | no route — `setup_prometheus`, called by `_setup_monitoring` | not a router; needs a home under `api/infrastructure/` |

These files run on the legacy stack (`api/helpers/_accesscontroller.py`, `api/schemas/`, `api/utils/dependencies.py`, `global_context.identity_access_manager`). Migrating one means porting it to the patterns in this file — do not carry its imports over, and do not copy them into new code.

The directory disappears once all three are moved and these go with them:

- `RouterName.CHAT` pointing at `api.endpoints.chat` (`api/utils/variables.py`)
- the `@TODO: legacy import` block registering `api.endpoints.admin` (`api/app.py`)
- `from api.endpoints.monitoring import setup_prometheus` (`api/app.py`)

`api/endpoints/me/` and `api/endpoints/proconnect/` are already empty — their contents were migrated.

If a task makes you touch one of these files, migrate it rather than extend it (principle 4).

---

## Adding a feature

```
- [ ] Domain entity + errors in api/domain/<context>/
- [ ] Repository port in api/domain/<context>/_<entity>repository.py — or query port + view for a read-only projection (see Query and view)
- [ ] SQL model in api/sql/models.py if the schema changes, then an Alembic revision: uv run alembic -c api/alembic.ini revision --autogenerate -m "<slug>" — test upgrade AND downgrade locally
- [ ] Postgres adapter in api/infrastructure/postgres/_postgres<entity>repository.py
- [ ] Use case in api/use_cases/<area>/_<verb><noun>usecase.py
- [ ] Export from domain/__init__.py, use_cases/__init__.py, postgres/__init__.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [etalab-ia/OpenGateLLM](https://github.com/etalab-ia/OpenGateLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
