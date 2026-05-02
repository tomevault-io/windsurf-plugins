---
trigger: always_on
description: Python development: modern Python, async/await, FastAPI, Django, data science, and ML patterns
---


# Python Development Patterns

Modern Python (3.10+) best practices for web development, data science, and general programming.

## Python Workflow

Before changing Python code:

```text
1. Read the existing project files first (`pyproject.toml`, `requirements.txt`, `setup.py`)
2. Check whether the repo already uses `uv`, `pip`, `poetry`, or another workflow
3. For new dependencies or version-sensitive work, verify current versions with the actual current date
4. Prefer the package manager and project conventions already in use
```

### CLI-First Python Development

Prefer CLI tools and existing repo conventions for package management:
```bash
# Virtual environment (recommended first step)
python3 -m venv .venv
source .venv/bin/activate  # Unix
.venv\Scripts\activate  # Windows

# Modern: uv (10-100x faster than pip)
uv venv
uv pip install fastapi uvicorn
uv pip sync requirements.txt

# Traditional: pip
pip install fastapi uvicorn
pip install -r requirements.txt
pip freeze > requirements.txt

# Framework CLIs (use these for scaffolding)
# Django
django-admin startproject myproject
python manage.py startapp myapp

# FastAPI with cookiecutter
pip install cookiecutter
cookiecutter https://github.com/tiangolo/full-stack-fastapi-template

# Poetry (if project uses it)
poetry init
poetry add fastapi
poetry install
```

### Post-Edit Verification

After meaningful Python changes, run the smallest useful check for the task:

```bash
# Common checks when configured in the repo
mypy src/
ruff check .
ruff format --check .
pytest
```

Scale up only when the change warrants it. Do not force every command after every tiny edit.

### Common Python Syntax Traps (Avoid These!)

```python
# WRONG: Mutable default argument
def append_to(item, target=[]):  # Bug! List is shared!
    target.append(item)
    return target

# CORRECT: Use None as default
def append_to(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target

# WRONG: Not using context managers for files
f = open('file.txt')
data = f.read()
f.close()  # Might not run if exception occurs!

# CORRECT: Always use with statement
with open('file.txt') as f:
    data = f.read()

# WRONG: Bare except clause
try:
    risky_operation()
except:  # Catches EVERYTHING including KeyboardInterrupt!
    pass

# CORRECT: Catch specific exceptions
try:
    risky_operation()
except (ValueError, TypeError) as e:
    logger.error(f"Operation failed: {e}")

# WRONG: Late binding in closures
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]  # Returns [2, 2, 2], not [0, 1, 2]!

# CORRECT: Capture value as default argument
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]  # Returns [0, 1, 2]

# WRONG: Using is for value comparison
if x is 1:  # Works sometimes, but wrong!
    pass

# CORRECT: Use == for values, is for identity
if x == 1:  # Value comparison
    pass
if x is None:  # Identity (OK for None, True, False)
    pass
```

### Python Version-Specific Features

Check Python version before using new features:

```python
# Python 3.10+ Pattern Matching
match command:
    case ["quit"]:
        return "Goodbye"
    case _:
        return "Unknown"

# Python 3.9+ Built-in Generic Types
def process(items: list[str]) -> dict[str, int]:
    pass

# Python 3.8+ Walrus Operator
if (n := len(data)) > 10:
    print(f"Processing {n} items")

# Check version in code if needed
import sys
if sys.version_info >= (3, 10):
    # Use match statement
else:
    # Use if/elif chain
```

---

## Modern Python Syntax

### Type Hints
```python
# Basic types
def greet(name: str) -> str:
    return f"Hello, {name}"

# Collections (Python 3.9+)
def process(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}

# Optional and Union
def find_user(id: int) -> User | None:
    return db.get(id)

# Generic types
from typing import TypeVar, Generic

T = TypeVar('T')

class Repository(Generic[T]):
    def get(self, id: int) -> T | None: ...
    def save(self, item: T) -> T: ...
```

### Pattern Matching (3.10+)
```python
match command:
    case ["quit"]:
        return "Goodbye"
    case ["load", filename]:
        return load_file(filename)
    case ["save", filename, *options]:
        return save_file(filename, options)
    case _:
        return "Unknown command"

# With guards
match user:
    case User(role="admin"):
        return full_access()
    case User(role="user", verified=True):
        return limited_access()
    case _:
        return no_access()
```

### Dataclasses
```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    id: int
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    tags: list[str] = field(default_factory=list)
    
    def __post_init__(self):
        self.email = self.email.lower()

# Immutable
@dataclass(frozen=True)
class Config:
    host: str
    port: int
```

---

## Async/Await Patterns

### Basic Async
```python
import asyncio

async def fetch_data(url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# Parallel execution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
