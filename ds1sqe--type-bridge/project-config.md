---
trigger: always_on
description: **type-bridge** is a Python ORM (Object-Relational Mapper) for TypeDB, designed to provide Pythonic abstractions over TypeDB's native TypeQL query language.
---

# Project Overview

**type-bridge** is a Python ORM (Object-Relational Mapper) for TypeDB, designed to provide Pythonic abstractions over TypeDB's native TypeQL query language.

TypeDB is a strongly-typed database with a unique type system that includes:

- **Entities**: Independent objects with attributes
- **Relations**: Connections between entities with role players
- **Attributes**: Values owned by entities and relations

## Python Version

This project requires **Python 3.13+** (see .python-version)

## Quick Start

```bash
# Install dependencies
uv sync --extra dev

# Install pre-commit hooks (required for development)
pre-commit install

# Run tests
uv run pytest                    # Unit tests (fast, no deps)
./test-integration.sh            # Integration tests (with Docker)

Podman users: integration tests work with Podman too—set `CONTAINER_TOOL=podman` (or `podman-compose`) so the fixtures call `podman compose ...` instead of Docker. Either runtime works as long as a compatible *compose* subcommand is available and the TypeDB container can start.

# Run examples
uv run python examples/basic/crud_01_define.py

# Code quality (run before committing)
uv run ruff check --fix .        # Lint and auto-fix
uv run ruff format .             # Format code
uvx ty check .                   # Type check library
uv run pyright tests/            # Type check tests
```

## Project Structure

```text
type_bridge/
├── __init__.py           # Main package exports
├── query.py              # TypeQL query builder
├── session.py            # Database connection and transaction management
├── reserved_words.py     # Reserved words validation
├── typedb_driver.py      # TypeDB driver wrapper
├── validation.py         # Validation utilities
├── attribute/            # Modular attribute system
│   ├── base.py           # Abstract Attribute base class
│   ├── string.py         # String attribute
│   ├── integer.py        # Integer attribute
│   ├── double.py         # Double attribute
│   ├── boolean.py        # Boolean attribute
│   ├── datetime.py       # DateTime attribute
│   ├── datetimetz.py     # DateTimeTZ attribute
│   ├── date.py           # Date attribute
│   ├── decimal.py        # Decimal attribute
│   ├── duration.py       # Duration attribute
│   └── flags.py          # Flag system (Key, Unique, Card, TypeFlags)
├── models/               # Base Entity and Relation classes (modularized)
│   ├── __init__.py       # Module exports
│   ├── base.py           # Base model functionality
│   ├── entity.py         # Entity class
│   ├── relation.py       # Relation class
│   ├── role.py           # Role definitions
│   └── utils.py          # Model utilities
├── expressions/          # Query expression system
│   ├── __init__.py
│   ├── aggregate.py      # Aggregation expressions
│   ├── base.py           # Base expression classes
│   ├── boolean.py        # Boolean expressions
│   ├── comparison.py     # Comparison expressions
│   ├── functions.py      # Function expressions
│   ├── role_player.py    # Role player expressions
│   └── string.py         # String expressions
├── fields/               # Field system
│   ├── __init__.py
│   ├── base.py           # Base field classes
│   └── role.py           # Role field definitions
├── crud/                 # CRUD operations (modularized)
│   ├── __init__.py       # Module exports (backward compatible)
│   ├── base.py           # Type variables (E, R)
│   ├── utils.py          # Shared utilities (format_value, is_multi_value_attribute)
│   ├── exceptions.py     # CRUD exceptions
│   ├── hooks.py          # Lifecycle hooks (CrudEvent, HookCancelled, CrudHook, HookRunner)
│   ├── entity/           # Entity CRUD operations
│   │   ├── __init__.py   # Entity module exports
│   │   ├── manager.py    # EntityManager class
│   │   ├── query.py      # EntityQuery chainable queries
│   │   └── group_by.py   # GroupByQuery for aggregations
│   └── relation/         # Relation CRUD operations
│       ├── __init__.py   # Relation module exports
│       ├── manager.py    # RelationManager class
│       ├── query.py      # RelationQuery chainable queries
│       ├── group_by.py   # RelationGroupByQuery for aggregations
│       └── lookup.py     # Relation lookup functionality
├── schema/               # Modular schema management
│   ├── __init__.py       # Module exports
│   ├── manager.py        # SchemaManager for schema operations
│   ├── info.py           # SchemaInfo container
│   ├── diff.py           # SchemaDiff for comparison
│   ├── migration.py      # MigrationManager for migrations
│   └── exceptions.py     # SchemaConflictError for conflict detection
└── generator/            # Code generator (TQL → Python)
    ├── __init__.py       # Public API: generate_models(), parse_tql_schema()
    ├── __main__.py       # CLI: python -m type_bridge.generator
    ├── models.py         # ParsedSchema, AttributeSpec, EntitySpec, RelationSpec
    ├── parser.py         # TypeQL parser with inheritance resolution
    ├── naming.py         # kebab-case → PascalCase/snake_case utilities
    ├── annotations.py    # Type annotations handling
    └── render/           # Code renderers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ds1sqe/type-bridge](https://github.com/ds1sqe/type-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
