---
trigger: always_on
description: This document summarizes the structure, purpose, and usage patterns found in the **enrichmcp** repository. The project provides a framework that exposes structured data models to AI agents via the Model Context Protocol (MCP). Below is a detailed look at the repository's key components, build instructions, examples, and development practices.
---

# Repository Overview: EnrichMCP

This document summarizes the structure, purpose, and usage patterns found in the **enrichmcp** repository. The project provides a framework that exposes structured data models to AI agents via the Model Context Protocol (MCP). Below is a detailed look at the repository's key components, build instructions, examples, and development practices.

## 1. Purpose and Scope

The README describes EnrichMCP as *"The ORM for AI Agents - Turn your data model into a semantic MCP layer"* and highlights its goals:
- generate typed tools from data models
- manage relationships between entities
- provide schema discovery for AI agents
- validate inputs and outputs using Pydantic
- support any backend data source

These points appear in the README between lines 11 and 21【F:README.md†L11-L21】.

The framework allows developers to define Pydantic models (entities) and relationships, register them with an `EnrichMCP` application, and automatically expose resources for AI consumption. It also offers optional SQLAlchemy integration to convert existing ORM models into EnrichMCP entities.

## 2. Project Layout

```
/ (repo root)
├── README.md          – introduction and quickstart
├── Makefile           – common development commands
├── pyproject.toml     – package metadata and tooling config
├── docs/              – user documentation (MkDocs site)
├── examples/          – runnable examples
├── src/enrichmcp/     – library implementation
└── tests/             – unit tests
```

### 2.1 Important Files
- `pyproject.toml` defines project metadata, required Python version, dependencies, optional dev tools, and tooling configuration including Ruff, Pyright, and coverage settings【F:pyproject.toml†L1-L159】.
- `Makefile` contains tasks for setup, linting, tests, docs, and CI usage. For example, running `make setup` creates a virtual environment and installs dependencies【F:Makefile†L18-L24】.
- `docs/` hosts Markdown guides describing core concepts, examples, pagination, and SQLAlchemy integration. The site is served via MkDocs.
- `src/enrichmcp/` implements the framework’s core logic. Modules include:
  - `app.py` – the main `EnrichMCP` application class.
  - `entity.py` – base `EnrichModel` providing serialization and description helpers.
  - `relationship.py` – descriptor for defining relationships and registering resolvers.
  - `pagination.py` – helper classes (`PageResult`, `CursorResult`, etc.).
  - `context.py` – thin wrapper around FastMCP’s context object.
  - `lifespan.py` – helper to combine async lifespans.
  - `sqlalchemy/` – optional SQLAlchemy integration utilities.
- `examples/` demonstrates usage patterns such as a hello world API, a shop API (in-memory and SQLite backed), an OpenAI chat agent, and SQLAlchemy integration.

## 3. Core Library

### 3.1 EnrichMCP Application
`src/enrichmcp/app.py` defines the `EnrichMCP` class. Important behaviors include:
- On initialization, it stores metadata and creates a `FastMCP` instance【F:src/enrichmcp/app.py†L41-L53】.
- Built‑in resource `explore_data_model` provides a comprehensive description of all registered entities, relationships, and usage hints【F:src/enrichmcp/app.py†L64-L98】.
- The `entity` decorator validates that models have descriptions and that all fields (except relationships) include `Field(..., description="...")`. Registered relationships are attached to the model class for later resolution【F:src/enrichmcp/app.py†L100-L173】.
- The `describe_model` method generates documentation describing each entity, its fields, and relationships【F:src/enrichmcp/app.py†L188-L263】.
- The `resource` decorator wraps functions as MCP resources, requiring descriptions and registering them with FastMCP【F:src/enrichmcp/app.py†L265-L325】.
- Before running the server, `run()` validates that all relationships have at least one resolver, raising a `ValueError` otherwise【F:src/enrichmcp/app.py†L327-L358】.

### 3.2 Entities and Relationships
`src/enrichmcp/entity.py` defines `EnrichModel`, a Pydantic BaseModel with additional helpers:
- Relationship fields are detected via `relationship_fields()` and removed from serialized output to prevent recursion【F:src/enrichmcp/entity.py†L31-L66】.
- The `describe()` method returns a Markdown formatted description including fields and relationships【F:src/enrichmcp/entity.py†L67-L128】.

`src/enrichmcp/relationship.py` implements the `Relationship` descriptor used to declare connections between entities. Key features:
- Supports resolver registration via `@Entity.field.resolver` which automatically creates MCP resources with descriptive names and docstrings【F:src/enrichmcp/relationship.py†L58-L105】.
- Validates that resolver return types match the annotated relationship type【F:src/enrichmcp/relationship.py†L112-L133】.

### 3.3 Pagination Utilities
`src/enrichmcp/pagination.py` defines paginated result types:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [featureform/enrichmcp](https://github.com/featureform/enrichmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
