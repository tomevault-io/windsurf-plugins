---
trigger: always_on
description: This is a SQLAlchemy serialization library that provides a mixin for easy model serialization.
---

# Cursor Rules for SQLAlchemy-serializer

## Project Context
This is a SQLAlchemy serialization library that provides a mixin for easy model serialization.

**Project Structure:**
- Main package: `sqlalchemy_serializer/`
- Core: `serializer.py` (SerializerMixin, Serializer classes)
- Library: `lib/schema.py` (Schema/Tree/Rule), `lib/fields.py` (field extraction), `lib/serializable/` (type serializers)
- Tests: `tests/` directory mirrors source structure

## Core Development Principles

**Code Quality & Standards**

Python code must follow PEP 8 strictly. Use type hints on all functions and class methods. Prefer explicit over implicit. Enforce minimal complexity—keep functions under 20 lines when feasible. Use descriptive variable names that reflect intent.

**Architecture Principles**
- Beautiful is better than ugly
- Explicit is better than implicit
- Simple is better than complex
- Complex is better than complicated
- Flat is better than nested
- Sparse is better than dense
- Keep functions small and elegant
- Readability counts

**Testing Requirements**

Write tests before or immediately after implementation. Use pytest for unit tests and pytest-cov for coverage tracking. Maintain minimum 80% code coverage on all modules. Test both success and failure paths. Mock external dependencies consistently.

**Project Testing Setup:**
- Run tests: `make test` (uses docker-compose)
- Run specific test: `make test file=tests/path/to/test.py`
- Test configuration: `pytest.ini_options` in `pyproject.toml` (verbose, color, log_cli)
- Coverage: `--cov=sqlalchemy_serializer --cov-report term-missing`
- Database: PostgreSQL container via docker-compose (session-scoped fixture)
- Test models: Defined in `tests/models.py` (FlatModel, NestedModel, RecursiveModel, CustomSerializerModel)

## Project-Specific Rules

**File Organization**

Keep modules focused—one responsibility per file. Create a `tests/` directory mirroring source structure. Store configuration in environment variables or dedicated config modules. Never commit `.env` files or secrets.

**Project Structure:**
- `sqlalchemy_serializer/serializer.py` - Core SerializerMixin and Serializer classes
- `sqlalchemy_serializer/lib/schema.py` - Schema tree for rule handling (greedy/strict modes)
- `sqlalchemy_serializer/lib/fields.py` - Field extraction utilities (SQL fields, properties)
- `sqlalchemy_serializer/lib/serializable/` - Type-specific serializers (datetime, date, time, decimal, enum, uuid, bytes)
- Each serializable type has its own module following the Base class pattern

**Dependencies & Imports**

Pin exact versions in `pyproject.toml`. Use virtual environments exclusively—no global installs. Review dependency licenses. Import standard library first, then third-party, then local modules. Remove unused imports automatically.

**Error Handling & Logging**

Never use bare `except:`. Log at appropriate levels (DEBUG, INFO, WARNING, ERROR). Include context in error messages. Use custom exceptions for domain-specific errors. Validate inputs at function entry points.

**Project-Specific:**
- Logger: `logging.getLogger("serializer")` with WARN level by default
- Custom exception: `IsNotSerializable` for unserializable types
- Use debug logging for serialization flow: `logger.debug("Serialize key:%s type:%s", ...)`

## Code Practices

**Performance & Maintenance**

Avoid mutable default arguments. Use generators for large datasets. Prefer built-in functions over loops. Document non-obvious logic inline. Keep functions pure when possible. Use pathlib for file operations instead of os.path.

**Database & Queries**

Use parameterized queries exclusively—no string concatenation. Leverage ORM features (SQLAlchemy) for common operations. Index frequently-queried columns. Document query intent with comments. Use database transactions appropriately.

**SQLAlchemy-Specific:**
- Use `sqlalchemy.inspect()` for model introspection (get mapper attributes)
- Access model fields via `getattr(model, key)` after validation
- Support SQLAlchemy 2.0+ patterns (declarative_base, relationship)
- Handle relationships automatically through SerializerMixin
- Test models use PostgreSQL with docker-compose for integration testing

**AI Assistance Configuration**

Provide complete file context using `@` mentions before multi-file refactors. Keep conversations focused—start fresh if conversation exceeds 15 exchanges. Use Agent mode for complex multi-step changes. Reference commit history for reverting problematic changes quickly.

## Workflow Rules

**Version Control**

Make atomic commits with clear messages. Keep working directory clean—commit before AI changes. Use branches for features and fixes. Squash commits before merging. Document rationale in PR descriptions, not just code.

**Review & Validation**

Review all AI-generated code—especially complex logic and security-sensitive code. Run full test suite locally before pushing. Use linting tools (ruff, pylint) on every change. Verify imports resolve correctly. Check for accidental debugging code or temporary variables.

## Common Patterns

**Serializer Architecture:**
- Callback chain pattern: `serialize_types` tuple checked in order (most specific first)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n0nSmoker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
