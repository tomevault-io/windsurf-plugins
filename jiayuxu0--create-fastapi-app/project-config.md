---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an enterprise-grade FastAPI backend template named "evoai-backend-template" with a clean three-layer architecture (API → Service → Repository → Model). It includes built-in RBAC permission management, user management, file management, audit logging, and other core enterprise features. The project uses UV for package management and focuses on providing a production-ready, scalable backend framework.

**Key Features:**
- 🏗️ **Clean Architecture**: Three-layer separation (API/Service/Repository)
- 🔐 **RBAC Authorization**: Role-based access control with menus and API permissions
- 👤 **User Management**: Complete user lifecycle with JWT authentication
- 📝 **Audit Logging**: Comprehensive activity tracking and monitoring
- 📁 **File Management**: Secure file upload/download with validation
- 🚀 **Performance**: Redis caching, database optimization, rate limiting
- 🐳 **Production Ready**: Docker support, health checks, monitoring
- 📖 **Documentation**: Auto-generated API docs with MkDocs integration

## Common Commands

### Environment Setup
```bash
# Install UV package manager (if not installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install project dependencies
uv sync

# Install development dependencies (includes testing, linting tools)
uv sync --dev

# Install documentation dependencies
uv sync --group docs
```

### Development Server
```bash
# Run development server with hot reload (entry point is src/__init__.py)
uv run uvicorn src:app --reload --host 0.0.0.0 --port 8000

# Run production server with multiple workers
uv run uvicorn src:app --host 0.0.0.0 --port 8000 --workers 4

# Alternative: Run via Python module
uv run python -m uvicorn src:app --reload
```

### Database Operations
```bash
# Initialize database (first time setup)
uv run aerich init-db

# Generate migration after model changes
uv run aerich migrate --name "describe_your_changes"

# Apply migrations
uv run aerich upgrade

# View migration history
uv run aerich history
```

### Testing
```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/test_users.py

# Run with coverage report
uv run pytest --cov=src --cov-report=html
```

### Code Quality

#### 🔧 Pre-commit Hooks (自动化)
```bash
# hooks 会在 uv sync 时自动安装并配置
# 每次 git commit 时自动运行，确保代码质量

# 手动运行所有检查
uv run pre-commit run --all-files

# 禁用 hooks (如不需要)
uv run pre-commit uninstall

# 跳过单次检查 (紧急提交)
git commit --no-verify -m "urgent fix"
```

#### ⚙️ 手动检查命令
```bash
# 代码检查和自动修复 (替代 black + isort)
uv run ruff check --fix src/

# 代码格式化
uv run ruff format src/

# 类型检查 (可选)
uv run mypy src/
```

📖 **详细配置**: 查看 [docs/pre-commit-guide.md](docs/pre-commit-guide.md)

### Docker Operations
```bash
# Build image
docker build -t backend-template .

# Run container
docker run -p 8000:8000 backend-template
```

### Documentation
```bash
# Install documentation dependencies
uv sync --group docs

# Serve documentation locally
uv run mkdocs serve

# Build documentation
uv run mkdocs build

# Deploy documentation to GitHub Pages
uv run mkdocs gh-deploy
```

## Architecture Overview

The project follows a clean three-layer architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                        API Layer                            │
│  (src/api/v1/) - Routes, parameter validation, responses    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      Service Layer                          │
│  (src/services/) - Business logic, permissions, validation  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   Repository Layer                          │
│  (src/repositories/) - Data access, CRUD operations         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      Model Layer                            │
│  (src/models/) - Tortoise ORM models, database schemas     │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Principles
- **Single Responsibility**: Each layer handles only its own logic
- **Dependency Injection**: Managed through FastAPI's dependency system
- **Type Safety**: Comprehensive Python type annotations throughout
- **Async First**: All I/O operations are asynchronous
- **Security First**: Multiple built-in security mechanisms

### Core Components

- **Authentication**: JWT-based with access tokens (4 hours) and refresh tokens (7 days)
- **Authorization**: RBAC system with roles, menus, and API permissions
- **Rate Limiting**: Built-in request rate limiting using SlowAPI
- **File Management**: Secure file upload/download with type validation and size limits
- **Audit Logging**: HTTP request logging and user activity tracking via middleware

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JiayuXu0/create-fastapi-app](https://github.com/JiayuXu0/create-fastapi-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
