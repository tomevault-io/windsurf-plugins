---
trigger: always_on
description: 这是一个使用 Python + FastAPI + SQLAlchemy + SQLite 构建的任务管理 REST API。
---

# Task Manager API — Agent 指导文件

## 项目概览

这是一个使用 Python + FastAPI + SQLAlchemy + SQLite 构建的任务管理 REST API。
该项目作为 OpenCode 的实战学习案例，演示如何在真实项目中使用 OpenCode 进行 AI 辅助开发。

## 架构

```text
src/
├── main.py      — 服务器入口，启动 FastAPI HTTP 服务
├── database.py  — SQLite 数据库初始化（SQLAlchemy）
├── models.py    — 数据库模型定义（tasks 表）
├── schemas.py   — Pydantic Schema 定义（请求/响应验证）
└── routes.py    — RESTful 路由处理（CRUD）
```

## 技术选型

- **Runtime**: Python 3.11+
- **HTTP Framework**: FastAPI
- **ORM**: SQLAlchemy 2.0
- **Database**: SQLite
- **Validation**: Pydantic
- **Language**: Python (type hints)
- **Linting/Formatting**: Ruff
- **Testing**: pytest + httpx

## Build/Lint/Test Commands

### Installation & Setup

```bash
# Install dependencies (development mode)
pip install -e .

# Install development dependencies
pip install -e ".[dev]"
```

### Linting & Formatting

```bash
# Run Ruff linter to check for errors
ruff check .

# Run Ruff linter with auto-fix
ruff check --fix .

# Run Ruff formatter
ruff format .

# Check import order and style (I category)
ruff check --select I .
```

### Type Checking

```bash
# Run mypy (if configured) – currently not set up
# mypy src/
```

### Testing

```bash
# Run all tests
pytest

# Run tests with verbose output
pytest -v

# Run a specific test file (when tests exist)
pytest tests/test_routes.py

# Run a single test by name
pytest -k "test_create_task"

# Run tests with coverage
pytest --cov=src

# Run tests in watch mode (requires pytest-watch)
ptw .
```

### Development Server

```bash
# Start development server with hot reload
uvicorn src.main:app --reload --port 3000

# Alternative: run via Python module
python -m src.main
```

### Database Operations

```bash
# Initialize/Recreate database tables
python -c "from src.database import init_db; init_db()"

# Delete database file (SQLite)
rm tasks.db
```

## Code Style Guidelines

### Naming Conventions

- **Variables & Functions**: `snake_case` (e.g., `get_db`, `task_id`, `row`)
- **Classes**: `PascalCase` (e.g., `Task`, `CreateTaskSchema`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `DATABASE_URL`)
- **Private Members**: Prefix with underscore `_private_method`
- **Module Names**: `snake_case` (e.g., `database.py`, `models.py`)

### Imports

- **Standard Library imports** first
- **Third-party imports** second
- **Local application imports** last
- Separate groups with a blank line
- Use absolute imports for local modules
- Avoid wildcard imports (`from module import *`)
- Example:

```python
from collections.abc import Generator
from datetime import datetime

from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from .models import Base
```

### Formatting

- **Line Length**: 100 characters (configured in Ruff)
- **Indentation**: 4 spaces per level (no tabs)
- **Quotes**: Use double quotes (`"`) for strings, single quotes (`'`) for characters within strings
- **Trailing Commas**: Include in multi-line collections
- **Blank Lines**:
  - Two blank lines before class/function definitions
  - One blank line between methods
  - Use blank lines to separate logical sections within functions

### Type Annotations

- Use type hints for all function parameters and return values
- Use `typing` module for complex types (e.g., `Optional`, `Union`, `List`, `Dict`)
- Leverage Python 3.11+ syntax (e.g., `str | None` instead of `Optional[str]`)
- Annotate instance variables in `__init__` method or class body
- Example:

```python
def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Error Handling

- Use specific exception types rather than bare `except:` clauses
- Raise `HTTPException` with appropriate status codes in route handlers
- Return descriptive error messages in JSON format: `{"error": "description"}`
- Use early returns to reduce nesting depth
- Validate input with Pydantic schemas before processing
- Example:

```python
task = db.query(Task).filter(Task.id == task_id).first()
if not task:
    raise HTTPException(status_code=404, detail="任务不存在")
```

### Database Conventions

- Model fields use `snake_case` (e.g., `created_at`, `updated_at`)
- Use SQLAlchemy 2.0's `Mapped` type annotations
- Define `__tablename__` explicitly (plural nouns)
- Provide `to_dict()` method for serialization
- Timestamp fields: `created_at` (default=`datetime.now`), `updated_at` (default=`datetime.now`, `onupdate=datetime.now`)
- Use `Base.metadata.create_all()` for table creation

### API Design

- **RESTful conventions**: Use plural resource names (`/api/tasks`)
- **HTTP Methods**: GET (retrieve), POST (create), PATCH (partial update), DELETE (remove)
- **Status Codes**:
  - 200 OK (successful GET/PATCH/DELETE)
  - 201 Created (successful POST)
  - 400 Bad Request (validation error)
  - 404 Not Found (resource doesn't exist)
- **Response Format**: Uniform JSON envelope:
  - Success: `{"data": ...}`
  - Error: `{"error": "description"}`
- **Request Validation**: Use Pydantic schemas for all request bodies
- **Documentation**: Include docstrings with OpenAPI descriptions

### Documentation

- Use docstrings for all public modules, classes, and functions
- Follow Google-style docstring format (brief description, Args, Returns, Raises)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ForceInjection/opencode-practise](https://github.com/ForceInjection/opencode-practise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
