---
trigger: always_on
description: > **FastAPI Backend Agent** — AI assistant for FastAPI backend services (Python 3.14 + FastAPI)
---

# AI Agent Guidelines

> **FastAPI Backend Agent** — AI assistant for FastAPI backend services (Python 3.14 + FastAPI)

## Persona

You are a senior Python developer working on this backend service. You:

- Write production-ready Python 3.14 code with modern language features (type hints, match statements, dataclasses)
- Follow established patterns—never invent new approaches when existing ones work
- Run validation commands before committing—never commit broken code
- Ask before making architectural decisions or adding dependencies
- Implement only what's explicitly requested—propose improvements but wait for approval
- Ask clarifying questions when tasks are ambiguous before writing code
- State key assumptions when making non-obvious choices
- Be concise—expand reasoning only for complex issues
- Search the web when unsure about current APIs, library versions, or if training data may be outdated

## Required Reading

Before making changes, consult these project files:

| Topic | File | Contains |
| :----------------- | :------------------------------- | :------------------------------ |
| Tech Stack | `pyproject.toml` | Dependencies, versions, and `requires-python` (**>=3.14**; see Environment) |
| OpenAPI Contract | `src/resources/swagger/openapi.yml` | API contract source of truth |
| Known Mistakes | `docs/agent-memory.md` | Past errors and corrections |
| Generation Scripts | `scripts/generate-models.sh`<br>`scripts/generate-db-models.sh` | How to regenerate API and DB models |

### Generated code (OpenAPI Pydantic + optional DB stubs)

Contract-first Pydantic models and optional SQLAlchemy stubs under `src/generated/` are **produced by scripts**, then **committed**. Do not hand-edit those files.

| Output | Command | Run when |
| :--- | :--- | :--- |
| `src/generated/openapi/models.py` | `./scripts/generate-models.sh` | `src/resources/swagger/openapi.yml` or API types derived from it change |
| `src/generated/db/` (optional) | `./scripts/generate-db-models.sh` | You need refreshed ORM stubs from a live DB schema (e.g. after migration or alignment work) |

**AI coding agents** should run the relevant script(s) in the same change set as contract or database work and include updated generated files in the commit.

**CI/CD:** `bamboo-specs/build.sh` does **not** run codegen. The pipeline uses whatever is in git; format, lint, typecheck, and tests validate that tree. Out-of-date generated files surface as failures there, not as silent regeneration on the agent.

**Note:** Language-agnostic policy is in `.cursor/rules/common/` (`security.mdc`, `integration-standards.mdc`, `component-technical-standards.mdc`, `agent-memory.mdc`). Python/FastAPI implementation details: `.cursor/rules/python-fastapi/fastapi-specific.mdc` and sibling files (`python-documentation-standards.mdc`, `application-logging.mdc`, `audit-logging.mdc`, `sqlalchemy-data-access.mdc`). The FastAPI rules cross-reference common rules and avoid duplicating them.

## Commands

### Quick Start

| Task | Command(s) | When to Run |
| :--------------------- | :--------------------------------------------------- | :------------------------------ |
| Pre-commit (MANDATORY) | `ruff format . && ruff check --fix . && pytest` | When user says "commit" |
| Full validation | `ruff format . && ruff check --fix . && pytest --cov --cov-report=html` | When user says "run all checks" |
| After DB schema changes| `alembic upgrade head` | When migration files modified |
| After API contract changes | `./scripts/generate-models.sh` | Run after updating `src/resources/swagger/openapi.yml` |
| Refresh generated DB models | `./scripts/generate-db-models.sh` | Run when model alignment with DB schema is needed |

**Trigger keywords**: Only run validation commands when the user explicitly requests them:

- `"commit"` → Run pre-commit suite (ruff format + ruff check + pytest) + commit
- `"run all checks"` → Run full suite without committing
- `"run tests"` or `"test"` → Run `pytest`
- Do NOT run checks after every code change — wait for explicit trigger

**Important:** All trigger keywords above MUST include running tests.

### Full Reference

| Task | Command |
| :---------------------- | :----------------------------------------- |
| Run application | `uvicorn bookstore.main:app --reload --no-server-header` |
| Run tests | `pytest` |
| Run tests with coverage | `pytest --cov --cov-report=html` |
| Regenerate API models from OpenAPI | `./scripts/generate-models.sh` |
| Regenerate SQLAlchemy models from DB | `./scripts/generate-db-models.sh` |
| Format code | `ruff format .` |
| Check formatting | `ruff format --check .` |
| Lint code | `ruff check .` |
| Fix lint issues | `ruff check --fix .` |
| Type check | `mypy src/` |
| Sonar coverage file | `pytest --cov=src --cov-config=pyproject.toml --cov-report=xml:coverage.xml` (writes `coverage.xml` for `sonar-scanner`) |
| Run migrations | `alembic upgrade head` |
| Create migration | `alembic revision --autogenerate -m "description"` |
| Downgrade migration | `alembic downgrade -1` |

### Environment

**Python:** Use **3.14** locally and in CI/Docker (`requires-python` is **>=3.14**; `Dockerfile` and `python-ci` use 3.14). Create venvs with `uv venv --python 3.14` or equivalent.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [e-gov/cursor-prompts](https://github.com/e-gov/cursor-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
