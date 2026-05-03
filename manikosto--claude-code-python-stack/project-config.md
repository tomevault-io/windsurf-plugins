---
trigger: always_on
description: This file provides guidance to Claude Code when working in Python projects that use this stack.
---

# CLAUDE.md

This file provides guidance to Claude Code when working in Python projects that use this stack.

## Project Type

This is a **Python QA & Development toolkit** — a collection of agents, skills, commands, hooks, and rules for building and testing production-grade Python applications.

## Mandatory Conventions

### Python Code

- Follow **PEP 8** strictly
- **Type annotations** on all public functions — no exceptions
- **Black** (88 chars) + **isort** + **ruff** for formatting/linting
- **mypy** in strict mode for type checking
- Use `logging` module, NEVER `print()` in production code
- Prefer `pathlib.Path` over `os.path`
- Use `f-strings` for string formatting
- Immutable by default: `@dataclass(frozen=True)`, `NamedTuple`, `tuple`
- Explicit `None` checks: `if value is None`, never `if value == None`
- Specific exceptions: never bare `except:`, always `except SpecificError`

### Test Code (STRICT OOP — Non-Negotiable)

- **ALL tests MUST be in classes** — no bare `def test_*()` functions, ever
- **ALL test classes MUST inherit from `BaseTest`** or a domain-specific base class
- **ALL API calls MUST go through service classes** (`UsersAPI`, `AuthAPI`, etc.) — never raw `httpx.get()` or `requests.post()` in tests
- **ALL API service methods MUST have `@allure.step`** decorators
- **ALL request/response bodies MUST use Pydantic models** — no raw dicts
- **ALL test data MUST come from `DataGenerator`/factories** — never hardcode emails, passwords, IDs
- **EVERY test follows Arrange-Act-Assert** pattern
- **Allure metadata is required on every test**:
  - `@allure.epic("...")` on class
  - `@allure.feature("...")` on class
  - `@allure.story("...")` on method
  - `@allure.severity(...)` on method
- **NEVER use `time.sleep()`** in tests — use `Waiter` with polling
- **Tests MUST be independent** — no shared mutable state, no execution order dependency

### Frameworks

- **FastAPI**: async-first, Pydantic Settings for config, `Depends()` for DI
- **Django**: split settings (base/dev/prod), custom User model, service layer pattern
- **SQLAlchemy 2.0**: `select()` style (not legacy `Query`), async sessions, `selectinload`/`joinedload`
- **Celery**: idempotent tasks, `task_acks_late=True`, batch by queue
- **pytest**: fixtures in conftest.py hierarchy, markers for test categorization

### Database

- **PostgreSQL**: `bigint` for IDs, `text` not `varchar(255)`, `timestamptz` not `timestamp`
- Always index foreign keys
- `CREATE INDEX CONCURRENTLY` for zero-downtime
- Parameterized queries only — never f-strings in SQL
- Django: `select_related`/`prefetch_related` to prevent N+1
- SQLAlchemy: `selectinload`/`joinedload` to prevent N+1

### Security

- Secrets via environment variables (`os.environ`, Pydantic `SecretStr`)
- `bandit -r src/` before every PR
- `pip-audit` for dependency vulnerabilities
- Never `eval()`, `exec()`, `pickle.loads()` with user data
- Never `yaml.load()` — use `yaml.safe_load()`
- Never `subprocess(shell=True)` with user input
- Django: `DEBUG=False` in production, CSRF enabled, HSTS headers

## Running Tests

```bash
# Run all tests with Allure
pytest --alluredir=allure-results -v

# Run with coverage
pytest --cov=src --cov-report=term-missing --alluredir=allure-results

# Run smoke suite
pytest -m smoke --alluredir=allure-results

# Run parallel
pytest -n auto --alluredir=allure-results

# Generate Allure report
allure serve allure-results

# Linting & type checking
ruff check .
black . --check
mypy .

# Security scan
bandit -r src/
pip-audit
```

## Project Structure Expected

```
project/
├── src/                        # Application source code
│   └── app/
│       ├── api/                # Routes/endpoints
│       ├── models/             # DB models (SQLAlchemy/Django)
│       ├── schemas/            # Pydantic schemas
│       ├── services/           # Business logic
│       ├── repositories/       # Data access layer
│       └── core/               # Config, security, database setup
├── tests/
│   ├── conftest.py             # Root fixtures (session-scoped)
│   ├── framework/              # Test framework (reusable)
│   │   ├── base/               # BaseTest, BaseAPIClient
│   │   ├── clients/            # HTTPClient wrappers
│   │   ├── models/             # Request/response Pydantic DTOs
│   │   ├── helpers/            # Assertions, DataGenerator, Allure helpers
│   │   └── config/             # Settings, endpoints
│   ├── api/                    # API service classes
│   │   ├── auth_api.py
│   │   └── users_api.py
│   └── tests/                  # Test classes
│       ├── test_auth/
│       ├── test_users/
│       └── test_orders/
├── alembic/                    # Migrations (if SQLAlchemy)
├── pyproject.toml
├── Dockerfile
└── docker-compose.yml
```

## Key Commands

| Command | Purpose |
|---------|---------|
| `/plan` | Create implementation plan before coding |
| `/tdd` | Test-driven development: RED → GREEN → REFACTOR |
| `/api-test` | Generate OOP API test suite with Allure |
| `/python-review` | Python code review (PEP 8, types, security) |
| `/qa-review` | Test code review (OOP, Allure, flaky patterns) |
| `/code-review` | Universal security & quality review |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manikosto/claude-code-python-stack](https://github.com/manikosto/claude-code-python-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
