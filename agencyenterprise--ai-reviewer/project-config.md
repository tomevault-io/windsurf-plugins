---
trigger: always_on
description: This is an AI-powered document review system with a Python backend (FastAPI + LangChain) and Next.js frontend. The system uses agent-based workflows to analyze documents for claims, citations, and references.
---

# Project Instructions

## Project Overview

This is an AI-powered document review system with a Python backend (FastAPI + LangChain) and Next.js frontend. The system uses agent-based workflows to analyze documents for claims, citations, and references.

---

## Quick Reference

### Running Commands

```bash
# Backend (always use uv)
uv run dev.py                    # Start full dev environment
uv run pytest                    # Run tests
uv run pytest tests/ -k "test"   # Run specific tests

# Frontend (always use pnpm)
cd frontend && pnpm install
cd frontend && pnpm dev
cd frontend && pnpm run openapi-generate  # Regenerate API types
```

---

## Python Backend Rules

### Use `uv` for running Python scripts

This project uses `uv` for Python dependency management and script execution. Always use `uv` instead of raw `python`, `pip`, or `pytest` commands.

```bash
# ✅ Correct - use uv run
uv run python script.py
uv run dev.py
uv run pytest tests/ -k "test_name"
uv add package-name

# ❌ Incorrect - don't use raw python or pip
python script.py
.venv/bin/python script.py
python -m pytest
pip install package-name
```

### Do NOT create `__init__.py` files

This codebase uses Python 3.3+ implicit namespace packages. Do not create `__init__.py` files in new directories - they are not used anywhere in `lib/` or `api/`.

```bash
# ✅ Correct - just create the module files
lib/workflows/new_workflow/
├── state.py
├── graph.py
├── manifest.py
└── nodes/
    └── process.py

# ❌ Incorrect - don't add __init__.py
lib/workflows/new_workflow/
├── __init__.py          # Don't create this
├── state.py
└── nodes/
    ├── __init__.py      # Don't create this either
    └── process.py
```

### Never use lazy imports inside methods

Always place imports at the top of the file. Lazy imports (imports inside functions or methods) are forbidden except when required to break a circular import or when there is an absolute technical necessity. Any such exception must be explained with a comment.

```python
# ✅ Correct - imports at the top of the file
from lib.services.users import UserService
from lib.models.document import Document

def process(doc: Document) -> None:
    service = UserService()
    ...

# ❌ Incorrect - lazy import inside a function
def process(doc):
    from lib.services.users import UserService  # don't do this
    service = UserService()
    ...

# ✅ Acceptable exception - circular import that cannot be resolved otherwise
def process(doc):
    # Imported here to avoid circular import between lib.services.users and lib.services.documents
    from lib.services.users import UserService
    service = UserService()
    ...
```

### Use Pydantic BaseModel, not dataclass

Always use Pydantic `BaseModel` for data models. Never use `@dataclass`.

```python
# ✅ Correct
from pydantic import BaseModel, Field

class DocumentMetadata(BaseModel):
    title: str = Field(description="Document title")
    page_count: int

# ❌ Incorrect
from dataclasses import dataclass

@dataclass
class DocumentMetadata:
    title: str
    page_count: int
```

### Use SQLAlchemy 2.0 style expressions

Always use SQLAlchemy 2.0 style `select()` statements, not the legacy 1.x `query()` pattern.

When referencing SQLModel columns in query expressions (`.where()`, `.filter()`, `.join()`, etc.), use the `col()` helper from SQLModel to ensure proper type checking.

```python
# ✅ Correct - SQLAlchemy 2.0 style with col() for type safety
from sqlalchemy import select
from sqlalchemy.orm import Session
from sqlmodel import col

def get_files(db: Session, project_id: uuid.UUID) -> Sequence[File]:
    stmt = select(File).where(col(File.project_id) == project_id)
    return db.execute(stmt).scalars().all()

def get_files_by_ids(db: Session, file_ids: list[uuid.UUID]) -> Sequence[File]:
    stmt = select(File).where(col(File.id).in_(file_ids))
    return db.execute(stmt).scalars().all()

# ❌ Incorrect - legacy 1.x style
def get_files(db: Session, project_id: uuid.UUID):
    return db.query(File).filter(File.project_id == project_id).all()

# ❌ Incorrect - missing col() causes type errors
def get_files(db: Session, project_id: uuid.UUID):
    stmt = select(File).where(File.project_id == project_id)  # mypy error: bool
    return db.execute(stmt).scalars().all()
```

### Database migrations - never run Alembic yourself

When making changes to database models in `lib/models/`:

- **Only modify the model file** - do not run migration commands
- **Remind the user** to run Alembic to generate and apply migrations
- The user will run: `uv run alembic revision --autogenerate -m "description"` and `uv run alembic upgrade head`

### Type Safety

```python
# Always use type hints
from typing import List, Optional, Union
from pydantic import BaseModel

def process_documents(files: List[File], config: Optional[ProcessingConfig] = None) -> DocumentResult:
    pass

# Use TypedDict for structured data (especially workflow state)
from typing import TypedDict
class WorkflowState(TypedDict, total=False):
    documents: List[Document]
    results: Optional[ProcessingResult]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agencyenterprise/ai-reviewer](https://github.com/agencyenterprise/ai-reviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
