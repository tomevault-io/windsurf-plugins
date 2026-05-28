---
trigger: always_on
description: | Type | Convention | Example |
---

# Code Standards & Style Guide

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Variables | `snake_case` | `user_count`, `total_items` |
| Constants | `UPPERCASE` | `MAX_RETRIES`, `API_BASE_URL` |
| Functions | `snake_case` | `get_user_data()`, `calculate_total()` |
| Classes | `PascalCase` | `UsercodeManager`, `DataProcessor` |
| Private/Internal | `_leading_underscore` | `_internal_helper()`, `_cache` |
| Ignored variables | `_` prefix | `for _ in range(10)`, `x, _ = get_pair()` |
| Module constants | `SCREAMING_SNAKE_CASE` | `DEFAULT_TIMEOUT = 30` |

## Code Structure

### Nesting
- **Maximum nesting depth**: 4 levels
- Use early returns, guard clauses, and extraction to reduce nesting
- Prefer flat over nested

```python
# Bad - too nested
def process(data):
    if data:
        if data.is_valid:
            if data.has_items:
                for item in data.items:
                    if item.active:
                        # deeply nested logic

# Good - use early returns
def process(data):
    if not data or not data.is_valid:
        return None
    if not data.has_items:
        return []
    return [item for item in data.items if item.active]
```

### Single Responsibility Principle

Every function and class should do **one thing well**. If you can't describe what it does in a single sentence without using "and", split it.

**Functions:**

```python
# Bad - does multiple things
def process_user(user_data: dict) -> None:
    # Validates, transforms, saves, AND sends email
    if not user_data.get("email"):
        raise ValueError("Missing email")
    user = User(**user_data)
    db.save(user)
    send_welcome_email(user.email)

# Good - each function has one job
def validate_user_data(user_data: dict) -> None:
    if not user_data.get("email"):
        raise ValueError("Missing email")

def create_user(user_data: dict) -> User:
    return User(**user_data)

def register_user(user_data: dict) -> User:
    validate_user_data(user_data)
    user = create_user(user_data)
    db.save(user)
    send_welcome_email(user.email)
    return user
```

**Classes:**

```python
# Bad - class does too much (God object)
class UserManager:
    def validate_user(self): ...
    def save_to_database(self): ...
    def send_email(self): ...
    def generate_report(self): ...
    def export_to_csv(self): ...

# Good - separate concerns
class UserValidator:
    def validate(self, user_data: dict) -> ValidationResult: ...

class UserRepository:
    def save(self, user: User) -> None: ...
    def find_by_id(self, user_id: int) -> User | None: ...

class UserNotifier:
    def send_welcome_email(self, user: User) -> None: ...
```

**How to tell if you're violating SRP:**

- Function name contains "and" or "or"
- Function has multiple reasons to change
- Hard to write a concise docstring
- Unit tests require complex setup
- You're passing unused parameters to satisfy different code paths

### Function Guidelines

- Keep functions under ~50 lines when practical
- Use descriptive names that indicate purpose
- Prefer pure functions where possible

### Line Length

- Maximum **88 characters** (black formatter default)
- Break long lines at logical points

## Type Hints (Strict Mode)

Type hints are **mandatory**, not optional. All code must be fully typed and pass strict type checking.

### Why Strict Typing?

- **IDE intelligence**: Enables autocompletion, refactoring, and jump-to-definition
- **Catch bugs early**: Static analyzers find type mismatches before runtime
- **Living documentation**: Types describe expected inputs/outputs without prose
- **Safer refactoring**: Type checkers catch breakages across the codebase
- **Better code review**: Reviewers immediately see data flow and contracts

### Requirements

- **All function parameters** must have type annotations
- **All return types** must be declared (including `-> None`)
- **Class attributes** must be annotated
- **Module-level variables** must be annotated when not immediately obvious
- **No `Any` type** unless absolutely unavoidable (requires justification comment)

### Strict Typing Rules

```python
# Bad - missing types
def fetch_user(user_id, include_metadata=False):
    ...

# Bad - implicit None return
def process_item(item: Item):
    print(item.name)

# Bad - using Any without justification
def handle_data(data: Any) -> Any:
    ...

# Good - fully typed
def fetch_user(user_id: int, include_metadata: bool = False) -> User | None:
    """Fetch user by ID."""
    ...

# Good - explicit None return
def process_item(item: Item) -> None:
    print(item.name)

# Acceptable - Any with justification
def handle_external_api_response(
    data: Any  # External API returns untyped JSON; validated below
) -> ProcessedData:
    validated = DataSchema.model_validate(data)
    return ProcessedData.from_schema(validated)
```

### Collection Types

Always use specific collection types, never bare `list`, `dict`, or `set`:

```python
# Bad - untyped collections
def get_users() -> list:
    ...

def get_config() -> dict:
    ...

# Good - parameterized collections
def get_users() -> list[User]:
    ...

def get_config() -> dict[str, int | str | bool]:
    ...

# Better - use TypedDict for structured dicts
from typing import TypedDict


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoobyFRS/GoobyDesk-Legacy](https://github.com/GoobyFRS/GoobyDesk-Legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
