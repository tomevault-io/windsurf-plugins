---
trigger: always_on
description: Architecture and development guidelines for Django Ninja backend
---


# Django Ninja Backend Architecture Guidelines

> For complete LLM context, see `.context/SYSTEM_PROMPT.md`, `.context/EXAMPLES.md`, and `.context/ANTI_PATTERNS.md`.
> This file is a summary — the `.context/` directory has full templates with worked examples.

## Tech Stack

- **Framework**: Django 5.2+ with Django Ninja
- **API Framework**: Django Ninja Extra (class-based controllers)
- **Authentication**: Django Ninja JWT
- **Database**: PostgreSQL
- **Cache**: Redis
- **Task Queue**: Celery with Redis broker
- **Package Manager**: UV for fast Python package management
- **Linting/Formatting**: Ruff (replaces Black, isort, Flake8)
- **Testing**: Pytest with Factory Boy

## Project Structure

### Modular App Structure

Each Django app should follow this structure:

```
app_name/
├── __init__.py
├── admin/                    # Admin configurations
│   ├── __init__.py
│   └── model_admin.py
├── controllers/              # API endpoints (class-based)
│   ├── __init__.py
│   └── model_controller.py
├── management/               # Management commands
│   ├── __init__.py
│   └── commands/
│       └── generate_data.py
├── migrations/               # Database migrations
├── models/                   # Database models
│   ├── __init__.py
│   ├── base.py
│   └── model.py
├── schemas/                  # Pydantic schemas
│   ├── __init__.py
│   └── model_schema.py
├── services/                 # Business logic layer
│   ├── __init__.py
│   └── model_service.py
├── tests/                    # Tests
│   ├── __init__.py
│   ├── conftest.py
│   ├── factories/
│   │   └── model_factory.py
│   └── test_model.py
├── apps.py
└── README.md
```

## Model Guidelines

### Base Model Usage

Always inherit from AbstractBaseModel or SoftDeleteModel:

```python
from core.models.base import AbstractBaseModel, SoftDeleteModel

# For models that need soft delete with custom managers
class MyModel(SoftDeleteModel):
    name = models.CharField(max_length=255)

    class Meta(SoftDeleteModel.Meta):
        pass

# For simpler models without custom managers
class SimpleModel(AbstractBaseModel):
    name = models.CharField(max_length=255)
```

### Model Best Practices

- Use UUID as primary key (provided by base models)
- Add `help_text` for all fields
- Add `db_index=True` for frequently queried fields
- Use `select_related` and `prefetch_related` appropriately
- Implement `__str__` method for all models
- Use `Meta.ordering` for default ordering
- Add database indexes for common queries

## Controller Guidelines

### Class-Based Controllers

Use Django Ninja Extra's `api_controller` decorator:

```python
from ninja_extra import api_controller, http_get, http_post, http_put, http_delete
from ninja_extra.pagination import paginate

from api.decorators import handle_exceptions, log_api_call

@api_controller("/items", tags=["Items"])
class ItemController:
    @paginate
    @http_get("/", response={200: list[ItemSchema]})
    @handle_exceptions()
    @log_api_call()
    def list_items(self, request, search: str | None = None):
        """List all items with optional search."""
        queryset = Item.objects.select_related("user").filter(user=request.user)
        if search:
            queryset = queryset.filter(name__icontains=search)
        return queryset

    @http_post("/", response={201: ItemSchema, 400: dict})
    @handle_exceptions()
    @log_api_call(include_payload=True)
    def create_item(self, request, payload: CreateItemSchema):
        """Create a new item."""
        item = Item.objects.create(user=request.user, **payload.model_dump())
        return 201, item
```

### Controller Best Practices

- Use descriptive docstrings for Swagger documentation
- Apply `@handle_exceptions()` decorator for error handling
- Apply `@log_api_call()` decorator for logging
- Use `@paginate` for list endpoints
- Return appropriate HTTP status codes
- Use type hints for all parameters

## Schema Guidelines

### Pydantic Schemas

Organize schemas by purpose:

```python
from ninja import Schema
from pydantic import EmailStr, Field

class ItemSchema(Schema):
    """Schema for item responses."""
    id: str
    name: str
    created_at: datetime

    class Config:
        from_attributes = True

class CreateItemSchema(Schema):
    """Schema for creating items."""
    name: str = Field(..., min_length=1, max_length=255)
    description: str | None = None

class UpdateItemSchema(Schema):
    """Schema for updating items."""
    name: str | None = None
    description: str | None = None
```

### Schema Best Practices

- Use `from_attributes = True` for ORM model conversion
- Add validation with Pydantic Field validators
- Separate Create, Update, and Response schemas
- Use Optional/None for optional fields
- Add docstrings for Swagger documentation

## Service Layer Guidelines

### Business Logic Separation

Use services for complex business logic:

```python
from core.services.base_service import CRUDService

class ItemService(CRUDService[Item]):
    model = Item

    def get_queryset(self):
        return super().get_queryset().select_related('user')

    def create_with_notifications(self, data: dict, user) -> Item:
        item = self.create(data, user=user)
        # Send notifications, trigger events, etc.
        return item


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattjaikaran/django-ninja-boilerplate](https://github.com/mattjaikaran/django-ninja-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
