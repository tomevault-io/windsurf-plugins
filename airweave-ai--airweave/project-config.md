---
trigger: always_on
description: Comprehensive guide for understanding and working with the CRUD layer in Airweave backend
---

# Airweave CRUD Layer Architecture

## Overview

The CRUD layer provides a consistent interface for database operations across all models in Airweave. It implements a sophisticated inheritance hierarchy that enforces proper access control, transaction management, and audit tracking.

## Inheritance Hierarchy

### Base Classes

#### 1. CRUDBaseOrganization (_base_organization.py)
- **Purpose**: For resources scoped to organizations (most common pattern)
- **Key Features**:
  - Enforces organization-level access control via `BaseContext`
  - Tracks user modifications with `created_by_email` and `modified_by_email`
  - Validates organization access on every operation
  - Supports both user and API key authentication contexts
  - Logger access via `ctx.logger` for contextual logging

#### 2. CRUDBaseUser (_base_user.py)
- **Purpose**: For pure user-level data (e.g., user profiles)
- **Key Features**:
  - Enforces strict user-level access (can only access own data)
  - No organization scoping
  - Simpler permission model - user can only CRUD their own resources

#### 3. CRUDPublic (_base_public.py)
- **Purpose**: For system-wide public resources (e.g., sources, destinations, embedding models)
- **Key Features**:
  - No access control - publicly accessible
  - Often used for system configuration data
  - Supports filtering by organization for multi-tenant scenarios
  - Includes `sync()` method for bulk updates

## Core Concepts

### BaseContext
The `BaseContext` (from `core.context`) is the universal context type for the CRUD layer. `ApiContext` and `SyncContext` both inherit from it:
```python
@dataclass
class BaseContext:
    organization: schemas.Organization  # Always present
    user: Optional[schemas.User] = None # Present for user auth, None for API keys/system
    logger: ContextualLogger            # Auto-derived from org/user if not provided
```

**Key Properties**:
- `has_user_context`: True if user is present (Auth0 or system with user)
- `tracking_email`: Returns user email for audit tracking
- `user_id`: Returns user UUID if available
- `has_feature(flag)`: Check organization feature flags
- `logger`: Contextual logger (auto-derived from identity, overridable)

`ApiContext(BaseContext)` adds HTTP-specific fields (`request_id`, `auth_method`, `analytics`).
`SyncContext(BaseContext)` adds sync-specific data (`sync_id`, `sync_job`, `collection`, etc.).

### Unit of Work Pattern
The `UnitOfWork` class manages database transactions:
```python
# Without UoW - auto-commits
await crud.create(db, obj_in=data, ctx=ctx)

# With UoW - manual transaction control
async with UnitOfWork(db) as uow:
    obj1 = await crud.create(db, obj_in=data1, ctx=ctx, uow=uow)
    obj2 = await crud.create(db, obj_in=data2, ctx=ctx, uow=uow)
    # Commits on context exit, rolls back on exception
```

## Common Patterns

### 1. Standard CRUD Operations
All base classes provide:
- `get(db, id, ctx)` - Get single resource
- `get_multi(db, ctx, skip, limit)` - Get multiple resources
- `create(db, obj_in, ctx, uow)` - Create resource
- `update(db, db_obj, obj_in, ctx, uow)` - Update resource
- `remove(db, id, ctx, uow)` - Delete resource

### 2. Access Control Validation
Organization-scoped resources validate access via:
```python
async def _validate_organization_access(ctx, organization_id):
    if ctx.has_user_context:
        # Check user has access to organization
    else:
        # Check API key belongs to organization
```

### 3. User Tracking
For organization-scoped resources with `track_user=True`:
- `created_by_email` and `modified_by_email` are automatically set
- API key operations set these to `None` (no user context)
- User operations set these to the authenticated user's email

### 4. Custom Methods
CRUD classes often extend base functionality:
```python
class CRUDSync(CRUDBaseOrganization):
    async def enrich_sync_with_connections(db, sync):
        # Custom method to load related data

    async def get(db, id, ctx, with_connections=True):
        # Override to add optional data loading
```

## Implementation Examples

### Simple Public Resource
```python
class CRUDEmbeddingModel(CRUDPublic[EmbeddingModel, EmbeddingModelCreate, EmbeddingModelUpdate]):
    pass

embedding_model = CRUDEmbeddingModel(EmbeddingModel)
```

### Organization-Scoped Resource
```python
class CRUDCollection(CRUDBaseOrganization[Collection, CollectionCreate, CollectionUpdate]):
    # Inherits all standard CRUD with org-level access control
    pass

collection = CRUDCollection(Collection)
```

### Complex Resource with Custom Logic
```python
class CRUDAPIKey(CRUDBaseOrganization[APIKey, APIKeyCreate, APIKeyUpdate]):
    async def create(self, db, *, obj_in, ctx, uow=None):
        # Generate secure key
        key = secrets.token_urlsafe(32)
        encrypted_key = credentials.encrypt({"key": key})

        # Use parent create with custom data
        return await super().create(
            db=db,
            obj_in={"encrypted_key": encrypted_key, ...},
            ctx=ctx,
            uow=uow
        )
```

### Special Cases
```python
class CRUDOrganization:
    # Doesn't inherit from base - organizations ARE the scope
    # Implements custom validation logic
    # Handles user-organization relationships

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
