---
trigger: always_on
description: You are an expert Python developer working on 'Dynantic'.
---

# 🤖 AI AGENT GUIDELINES

You are an expert Python developer working on 'Dynantic'.
You must write production-ready code. Dynantic is a library for interacting with DynamoDB using Python so it must be fast and easy to use and the code must satisfy the needs of a production environment.
Follow these rules strictly when debugging.
Use uv to run commands.
To run integration tests you first need to launch the localstack docker container.
You can do this by running `docker compose up -d` in the root directory of the project.
After every changes to the code you need to update the README.md file and the AGENTS.md file if something changed.

## 1. Code Structure & Architecture

### 📦 Module Overview
The `dynantic/` package follows a **layered architecture** pattern:

```
dynantic/
├── __init__.py          # Public API exports (DynamoModel, Key, SortKey)
├── base.py              # Core model class + CRUD operations
├── config.py            # Configuration dataclass
├── fields.py            # Field decorators (Key, SortKey, GSIKey, etc.)
├── query.py             # Query Builder pattern implementation
├── scan.py              # Scan Builder pattern implementation
├── conditions.py        # Condition DSL for filters and conditional writes
├── updates.py           # Update Builder for atomic operations
├── serializer.py        # Type conversion middleware
└── exceptions.py        # Custom exception types
```

### 🏗️ Architecture Layers

**1. Meta-Programming Layer (base.py - DynamoMeta)**
- **Purpose**: Introspects class definitions at import time (NOT runtime)
- **Responsibilities**:
  - Validates presence of `class Meta` with `table_name`
  - Scans fields to identify Primary Key (`_dynamo_pk`) and Sort Key (`_dynamo_sk`)
  - Builds and injects `ModelOptions` into class as `_meta` attribute
  - Raises errors if schema is invalid (e.g., missing PK)

**2. Model Layer (base.py - DynamoModel)**
- **Purpose**: User-facing base class that combines Pydantic validation + DynamoDB operations
- **Key Methods**:
  - **Class Methods** (operate without instances):
    - `.get(pk, sk=None) -> T | None` - Fetch single item
    - `.delete(pk, sk=None)` - Delete without fetching
    - `.scan() -> Iterator[T]` - Paginated full-table scan
    - `.query(pk_val) -> DynamoQueryBuilder[T]` - Start query chain
  - **Instance Methods**:
    - `.save()` - Persist model to DynamoDB
    - `.delete_item()` - Delete current instance
- **Design Patterns**:
  - Singleton pattern for `_client` (lazy-initialized)
  - Generic TypeVar `T` for type-safe returns

**3. Query Builder Layer (query.py - DynamoQueryBuilder)**
- **Purpose**: Fluent API for constructing DynamoDB queries with key conditions
- **Pattern**: Builder Pattern with method chaining
- **Methods**:
  - `.starts_with(prefix)` - Sort Key condition
  - `.equals(value)` - Exact Sort Key match
  - `.between(start, end)` - Range condition
  - `.filter(condition)` - **NEW**: Filter on non-key attributes
  - `.limit(n)` - Max results
  - `.reverse()` - Descending order
  - `.all() -> list[T]` - Execute and collect all
  - `.__iter__()` - Paginated generator execution
- **Key Insight**: Builds `KeyConditionExpression` and optional `FilterExpression` incrementally

**4. Scan Builder Layer (scan.py - DynamoScanBuilder)**
- **Purpose**: Fluent API for table/index scans without key conditions
- **Pattern**: Builder Pattern with method chaining (similar to Query Builder)
- **Methods**:
  - `.filter(condition)` - Filter results on any attributes
  - `.limit(n)` - Max results
  - `.using_index(name)` - Scan a GSI instead of main table
  - `.all() -> list[T]` - Execute and collect all
  - `.first() -> T | None` - Get first result
  - `.page(start_key) -> PageResult[T]` - Paginated execution
- **Key Feature**: Automatically merges discriminator filters for polymorphic models
- **Performance Note**: Scans are expensive - filters don't reduce RCU consumption

**5. Serialization Layer (serializer.py - DynamoSerializer)**
- **Purpose**: Middleware to bridge Python types ↔ DynamoDB JSON format
- **Critical Functions**:
  - `.to_dynamo(dict)` - Python dict → DynamoDB JSON (`{"S": "...", "N": "..."}`)
  - `.to_dynamo_value(value)` - Single value → DynamoDB format (for ExpressionAttributeValues)
  - `.from_dynamo(item)` - DynamoDB JSON → Python dict
- **Type Conversions**:
  - `float` → `Decimal` (before sending to Boto3)
  - `Decimal` → `float` (after receiving from DynamoDB)
  - **WHY**: Boto3 TypeSerializer rejects floats; DynamoDB requires Decimals for precision

**5. Field Marker Layer (fields.py)**
- **Purpose**: Mark fields as DynamoDB keys without explicit config
- **Mechanism**: Injects metadata flags into Pydantic's `json_schema_extra`
- **Functions**:
  - `Key()` - Marks field as Partition Key (sets `_dynamo_pk=True`)
  - `SortKey()` - Marks field as Sort Key (sets `_dynamo_sk=True`)

**6. Configuration Layer (config.py - ModelOptions)**
- **Purpose**: Data Transfer Object for model metadata
- **Fields**:
  - `table_name: str` (required)
  - `pk_name: str` (discovered by metaclass)
  - `sk_name: str | None` (optional)
  - `region: str` (default: "eu-west-1")

### 🔄 Request Flow Example
```python
# 1. User defines model
class User(DynamoModel):
    email: str = Key()        # fields.py injects flag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Simi24/dynantic](https://github.com/Simi24/dynantic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
