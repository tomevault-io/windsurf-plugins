---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

This project uses a comprehensive Makefile for development tasks:

### Primary Development Commands
- `make setup` - Complete one-time setup: creates venv, installs dependencies, sets up Docker services, and starts the server
- `make run` - Start Django development server (generates test JWTs first)
- `make jwt EMAIL=user@example.com` - Get JWT access and refresh tokens for a specific user
- `make check` - Run all code quality checks (format, lint, mypy, migration-check, i18n-check, file-length)
- `make test` - Run pytest test suite in parallel with coverage reporting. On failure, saves the failures section to `.tests.output`
- `make test-linear` - Run pytest test suite sequentially (single process)
- `make test-failed` - Re-run only failed tests

### Code Quality & Formatting
- `make format` - Auto-format code with ruff
- `make lint` - Run linting with ruff (auto-fixes issues)
- `make mypy` - Run strict type checking with mypy
- `make i18n-check` - Verify translation files are compiled and up-to-date

### Internationalization (i18n)
- `make makemessages` - Extract translatable strings and update .po files
- `make compilemessages` - Compile .po files into .mo binaries (commit after running!)
- `make i18n-check` - Verify .mo files are up-to-date with .po files

### Database Management
- `make migrations` - Create new Django migrations
- `make migrate` - Apply pending migrations
- `make bootstrap` - Initialize database with base data
- `make seed` - Populate database with test data
- `make nuke-db` - **DESTRUCTIVE**: Reset database and regenerate migration files (preserves special data migrations)
- `make restart` - **DESTRUCTIVE**: Deletes all migration files, regenerates them, restarts Docker, and bootstraps from scratch

### Background Services
- `make run-celery` - Start Celery worker for background tasks
- `make run-celery-beat` - Start Celery beat scheduler
- `make run-flower` - Start Flower (Celery monitoring UI)

### Testing Variants
- `make test-linear` - Run tests sequentially (single process, useful for debugging)

## Project Architecture

Revel is a Django-based event management platform with the following structure:

### Core Applications
- **accounts/** - User authentication, registration, JWT handling, GDPR compliance
- **events/** - Core event management, organizations, tickets, memberships, invitations
- **questionnaires/** - Dynamic questionnaire system with LLM-powered evaluation
- **notifications/** - Multi-channel notification system (in-app, email, Telegram) with user preferences and digest support
- **wallet/** - Apple Wallet pass generation for event tickets (.pkpass files)
- **geo/** - Geolocation features, city data, IP-based location detection
- **telegram/** - Telegram bot integration with FSM-based conversation flows
- **api/** - Global API configuration, exception handlers, rate limiting
- **common/** - Shared utilities, authentication, base models, admin customizations

### Key Architectural Patterns

#### Service Layer Pattern
Business logic is encapsulated in service modules:
- `events/service/` - Event management, organization services
- `accounts/service/` - User management, authentication services
- `questionnaires/service/` - Questionnaire evaluation logic
- `notifications/service/` - Notification dispatching, eligibility, digests, reminders

#### Controller Pattern (Django Ninja)
API endpoints are organized in controller classes:
- Controllers inherit from `UserAwareController` for common user/auth functionality
- Use `@api_controller` decorator with consistent tagging
- Implement pagination with `PageNumberPaginationExtra`
- Support search with `@searching` decorator

#### Permission System
- Custom permission classes in `events/controllers/permissions.py`
- Organization-based access control with roles (Owner, Staff, Member)
- Event-level permissions with eligibility checking
- JWT-based authentication with optional anonymous access

### Technology Stack
- **Backend**: Django 5+ with Django Ninja for API
- **Database**: PostgreSQL with PostGIS for geo features
- **Async Tasks**: Celery with Redis
- **Authentication**: JWT with custom user model
- **File Storage**: Local filesystem in development; configurable for production
- **Containerization**: Docker Compose for development services

### Testing Framework
- **pytest** with Django integration
- Test data generation via ORM and shared fixtures in `conftest.py`
- Coverage reporting with HTML output
- Celery task testing with pytest-celery

### Code Quality Standards
- **Formatting**: ruff (replaces black)
- **Linting**: ruff with Django-specific rules
- **Type Checking**: mypy with strict settings and Django plugin
- **Docstrings**: Google-style format required
- **Coverage**: 90%+ branch coverage required in CI pipeline

### Development Environment
- Python 3.13+ required
- **UV for dependency management** (use `uv add`/`uv remove` - NEVER use pip directly)
- Docker for services (PostgreSQL, Redis, ClamAV)
- Virtual environment automatically created in `.venv/`

### Key Configuration Files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letsrevel/revel-backend](https://github.com/letsrevel/revel-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
