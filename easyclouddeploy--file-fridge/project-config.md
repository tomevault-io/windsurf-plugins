---
trigger: always_on
description: This document provides guidelines for **AI agents** (Claude Code, GitHub Copilot, Cursor, Aider, etc.) working with the File Fridge codebase. It complements `CLAUDE.md` with agent-specific workflows, best practices, and automation guidelines.
---

# AGENTS.md - AI Agent Guide for File Fridge

## Overview

This document provides guidelines for **AI agents** (Claude Code, GitHub Copilot, Cursor, Aider, etc.) working with the File Fridge codebase. It complements `CLAUDE.md` with agent-specific workflows, best practices, and automation guidelines.

**Target Audience:** AI coding assistants, autonomous agents, and AI-powered development tools

---

## Quick Reference for Agents

**Project Type:** UV-based Python FastAPI application
**Agent-Friendly Features:**
- ✅ Comprehensive API documentation at `/docs` (Swagger)
- ✅ Type hints throughout codebase (Pydantic schemas)
- ✅ Clear separation of concerns (routers → services → database)
- ✅ Auto-formatted with Black (100 chars)
- ✅ Linted with Ruff (extensive rules)
- ✅ Well-documented models and schemas

**Key Files for Context:**
- `CLAUDE.md` - Comprehensive codebase documentation
- `README.md` - User-facing documentation
- `app/models.py` - Database schema (24 models)
- `app/schemas.py` - API contracts (Pydantic)
- `pyproject.toml` - Project metadata and dependencies

---

## Critical Agent Rules

### 🚨 Non-Negotiable Requirements

**1. Package Manager - UV Only**
```bash
# ✅ CORRECT - Use UV for all Python operations
uv run uvicorn app.main:app --reload
uv run pytest
uv run alembic upgrade head

# ❌ WRONG - Never use pip
pip install package-name  # FORBIDDEN
python -m pip install     # FORBIDDEN
```

**2. Database Changes - SQLAlchemy Auto-Sync First**

**Simple schema changes use SQLAlchemy `create_all()` (preferred):**
- ✅ Adding new tables
- ✅ Adding new columns to existing tables  
- ✅ Adding new indexes
- ✅ Creating relationships/foreign keys

**Complex migrations use Alembic (when needed):**
- 🔧 Column type conversions (e.g., INTEGER → VARCHAR)
- 🔧 Column/table renames
- 🔧 Data migrations or backfills
- 🔧 Multi-step schema changes
- 🔧 Changes not supported by SQLite's `ALTER TABLE`

```bash
# ✅ For simple changes (new tables/columns/indexes):
# Just update models.py - SQLAlchemy auto-sync handles it on next startup
uv run python -c "from app.database import init_db; init_db()"  # Test locally

# ✅ For complex migrations only:
uv run alembic revision --autogenerate -m "Convert column type"
uv run alembic upgrade head
```

**Why this approach?**
- SQLite supports `CREATE TABLE` and `ALTER TABLE ADD COLUMN` natively
- SQLAlchemy's `create_all()` is idempotent and handles these automatically
- Faster iteration during development
- Alembic reserved for operations SQLite cannot handle natively

**3. Code Quality - Format and Lint**
```bash
# ✅ Run before committing
uv run black app/
uv run ruff check app/ --fix

# Agent should auto-format code to match project standards
```

---

## Agent Workflows

### Workflow 1: Adding a New Feature

**Context Gathering:**
1. Read `CLAUDE.md` for architecture overview
2. Check `app/models.py` for relevant database models
3. Review `app/schemas.py` for existing API contracts
4. Examine similar endpoints in `app/routers/api/`

**Implementation Steps:**
1. **Design**: Plan the feature (models, API, services)
2. **Models**: Update `app/models.py` if database changes needed
3. **Test Schema Changes** (if step 2 applies):
   ```bash
   # For new tables/columns/indexes - SQLAlchemy auto-sync
   uv run python -c "from app.database import init_db; init_db()"
   
   # For complex changes (renames, type conversions) - Alembic
   uv run alembic revision --autogenerate -m "Add feature X tables"
   uv run alembic upgrade head
   ```
4. **Schemas**: Add Pydantic schemas to `app/schemas.py`
5. **Service**: Implement business logic in `app/services/`
6. **Router**: Add API endpoints in `app/routers/api/`
7. **Register**: Include router in `app/main.py`
8. **Format**: Run Black and Ruff
9. **Test**: Manual testing or add pytest tests
10. **Commit**: Clear, descriptive commit message

**Example Agent Prompt Response:**
```
I'll add the new user authentication feature. Here's my implementation plan:

1. Database changes needed:
   - New table: User (id, username, password_hash, created_at)
   - New table: Session (id, user_id, token, expires_at)

2. Creating migration:
   [Shows alembic command]

3. Adding Pydantic schemas:
   - UserCreate, UserResponse, LoginRequest, LoginResponse

4. Implementing auth service:
   - Password hashing with bcrypt
   - Token generation with secrets

5. Adding API endpoints:
   - POST /api/v1/auth/register
   - POST /api/v1/auth/login
   - GET /api/v1/auth/me

[Proceeds with implementation...]
```

### Workflow 2: Debugging an Issue

**Context Gathering:**
1. Read error message/stack trace completely
2. Locate relevant code files
3. Check recent git commits for related changes
4. Review database schema if DB-related
5. Check logs in `data/` directory

**Investigation Steps:**
1. **Reproduce**: Understand how to trigger the issue
2. **Isolate**: Identify the specific function/service causing it
3. **Root Cause**: Trace back to the source
4. **Fix**: Implement minimal change to resolve
5. **Verify**: Test the fix doesn't break other functionality

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EasyCloudDeploy/file-fridge](https://github.com/EasyCloudDeploy/file-fridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
