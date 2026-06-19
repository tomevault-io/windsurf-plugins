---
trigger: always_on
description: Production-grade conventions for FastAPI projects. Follow these strictly when generating or reviewing code. Examples use SQLAlchemy + Celery but every principle applies regardless of ORM or task queue.
---

# FastAPI Production Rules

Production-grade conventions for FastAPI projects. Follow these strictly when generating or reviewing code. Examples use SQLAlchemy + Celery but every principle applies regardless of ORM or task queue.

---

## Toolchain

Always use the Astral stack. Never use pip, black, flake8, isort, or mypy directly.

### Package manager — uv

```bash
uv sync                    # install all dependencies including dev
uv sync --no-dev           # production only
uv add package-name        # add production dependency
uv add --dev package-name  # add dev dependency
uv remove package-name     # remove dependency
uv run pytest              # run any command in project environment
```

Never use `pip install` in a uv project. Always commit `uv.lock`.

### pyproject.toml structure

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12,<3.13"
dependencies = [
  "fastapi[standard]>=0.135.1",
  "pydantic>=2.0.0",
  "pydantic-settings>=2.0.0",
  "structlog>=25.5.0",
  "uvicorn[standard]>=0.30.0",
]

[dependency-groups]
dev = [
  "pytest>=8.0.0",
  "pytest-asyncio>=0.24.0",
  "ruff>=0.8.0",
  "ty==0.0.12",
  "pytest-cov>=7.1.0",
]

[tool.ruff]
target-version = "py312"
line-length = 99

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "F",   # pyflakes
    "I",   # isort
    "UP",  # pyupgrade — modernizes Python syntax automatically
    "B",   # flake8-bugbear — catches common bugs
    "S",   # flake8-bandit — security checks
    "T20", # flake8-print — catches print() statements
]

[tool.ruff.lint.per-file-ignores]
"tests/*" = ["S101", "S106", "S108", "S105"]
"scripts/*" = ["T201", "T203"]

[tool.ty.src]

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

### Linter and formatter — ruff

```bash
uv run ruff check . --fix  # lint and autofix
uv run ruff format .       # format
uv run ruff format . --check  # CI check without modifying
```

`--fix` automatically removes unused imports, upgrades syntax, corrects import order. Always run before committing.

### Type checker — ty

```bash
uv run ty check app/
```

ty is Astral's type checker written in Rust. Use it instead of mypy or pyright.

### Pre-commit checklist

Run in this order before every commit:

```bash
uv run ruff check . --fix
uv run ruff format .
uv run ty check .
uv run pytest
```

---

## Project Structure

```
app/
├── core/
│   ├── config.py           # Settings via pydantic-settings
│   ├── dependencies.py     # Type aliases: CurrentUser, CurrentAdmin, CurrentOwner
│   ├── constants.py        # Values that never change between environments
│   ├── db/database.py      # DB engine and session factory
│   ├── security/auth.py    # JWT, password hashing, magic links
│   ├── http/exceptions.py  # AppError base class and subclasses
│   ├── http/responses.py   # DataResponse, PaginatedResponse, MessageResponse
│   ├── logging/logger.py   # structlog setup
│   ├── limiter.py          # Rate limiter singleton
│   ├── storage.py          # Storage abstraction (local/S3)
│   └── mail/mailer.py      # Email sending
│
└── [domain]/               # tickets/, auth/, users/, billing/, etc.
    ├── router.py           # HTTP endpoints only
    ├── service.py          # Business logic — pure Python, no FastAPI imports
    ├── model.py            # SQLAlchemy models
    └── schemas.py          # Pydantic I/O schemas
```

---

## Rule 1 — Router → Service → Model

The router is an HTTP translator. It validates input, calls a service, and returns a response. Nothing else.

The service is pure Python. It contains all business logic and never imports FastAPI, HTTPException, Request, or Depends.

```python
# CORRECT — router.py
@router.post("/tickets")
async def create_ticket(
    data: TicketCreate,
    user: CurrentUser,
    db: AsyncSession = Depends(get_db),
) -> DataResponse[TicketRead]:
    ticket = await ticket_service.create_ticket(db, data, user)
    return DataResponse(data=TicketRead.model_validate(ticket))
```

```python
# CORRECT — service.py (no FastAPI imports)
async def create_ticket(db: AsyncSession, data: TicketCreate, user: User) -> Ticket:
    ticket = Ticket(
        title=data.title,
        organization_id=user.organization_id,
        created_by=user.id,
    )
    db.add(ticket)
    await db.commit()
    await db.refresh(ticket)
    logger.info("ticket.created", ticket_id=str(ticket.id))
    return ticket
```

```python
# WRONG — logic in router
@router.post("/tickets")
async def create_ticket(data: TicketCreate, user: CurrentUser, db: AsyncSession = Depends(get_db)):
    ticket = Ticket(title=data.title, organization_id=user.organization_id)
    db.add(ticket)
    await db.commit()  # DB logic belongs in service
    return {"id": str(ticket.id)}
```

**Never in a router:**
- Direct database access
- Business logic or conditionals
- Private helper functions (`_something`)
- Manual data mappings (put these in schemas)

---

## Rule 2 — Use Annotated for Dependencies

```python
# dependencies.py
CurrentUser = Annotated[User, Depends(get_current_user)]
CurrentAdmin = Annotated[User, Depends(require_admin)]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [17tayyy/fastapi-production-rules](https://github.com/17tayyy/fastapi-production-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
