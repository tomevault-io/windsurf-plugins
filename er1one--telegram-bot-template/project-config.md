---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a production-ready Telegram bot template built with:
- **aiogram 3.x** - Asynchronous Telegram Bot API
- **FastAPI** - Webhook endpoint for receiving updates
- **PostgreSQL + Tortoise ORM** - Persistent data storage with migrations (Aerich)
- **Redis** - FSM state storage and caching
- **Docker Compose** - Containerized dev/prod environments
- **aiogram-i18n + Fluent** - Multi-language support
- **Python 3.13+** - Minimum required version

The bot operates exclusively via **webhook mode** (not polling), requiring a public URL for receiving updates from Telegram.

## Quick Start Commands

All development commands use Docker Compose and `make`:

```bash
# Switch to development environment and start
make dev

# Switch to production environment and start
make prod

# Container management
make build      # Build images
make up         # Start containers
make down       # Stop containers
make ps         # Show container status
make logs [service]   # View logs (omit service name for all)
make shell SERVICE    # Enter a container shell (e.g., make shell bot)
make rebuild    # Rebuild and restart everything

# Database migrations (Aerich)
make aerich-init       # Initialize Aerich (run once)
make aerich migrate    # Create migration after model changes
make aerich upgrade    # Apply pending migrations
make aerich downgrade  # Rollback last migration
make aerich history    # View migration history

# Database backup
make db-backup   # Create backup in ./backups/

# Cleanup
make clean      # Remove current environment containers and volumes
make clean-all  # Remove both dev and prod environments completely
```

## Architecture Overview

### Entry Point
**[bot/main.py](bot/main.py)** - FastAPI app initialization, webhook setup, middleware registration, and event handlers (startup/shutdown)

### Core Configuration
- **[bot/core/config.py](bot/core/config.py)** - Settings class using pydantic, loads from `.env`, provides `tortoise_url` and `redis_url` properties
- **[bot/core/loader.py](bot/core/loader.py)** - Initializes FastAPI app, Bot instance, Dispatcher, RedisStorage, and Tortoise ORM configuration
- **[bot/core/logging.py](bot/core/logging.py)** - Loguru configuration for structured logging

### Request Flow Architecture

```
Telegram Updates
    ↓
FastAPI Webhook Endpoint [bot/routes/]
    ↓
Dispatcher (aiogram)
    ↓
Middlewares [bot/middlewares/]
    - UserRegistrationMiddleware: Registers users in DB on first interaction
    - i18n middleware: Sets user language from cache or detects it
    ↓
Routers & Handlers [bot/handlers/]
    ├─ private/ - Direct message handlers
    └─ groups/ - Group message handlers
    ↓
Filters [bot/filters/] - Custom filter logic applied at router level
    ↓
Services [bot/services/] - Business logic (UserService, BroadcastService)
    ↓
Managers [bot/managers/] - Direct DB/Redis/i18n access
    ↓
Models [bot/models/] - Tortoise ORM database models

State Management: Redis FSM storage via RedisStorage
```

### Directory Structure

- **[bot/handlers/](bot/handlers/)** - Command and message routers
  - `private/` - Private chat handlers
  - `groups/` - Group chat handlers
  - `errors_router.py` - Error logging handler (conditionally loaded)

- **[bot/filters/](bot/filters/)** - Custom filters (e.g., IsPrivateChat, IsAdmin, IsChatAdmin, HasMedia)

- **[bot/keyboards/](bot/keyboards/)** - Inline keyboard builders

- **[bot/middlewares/](bot/middlewares/)** - Global middlewares registered at dispatcher level
  - UserRegistrationMiddleware - Auto-registers users in DB
  - i18n middleware - Language detection and caching

- **[bot/models/](bot/models/)** - Tortoise ORM models (User, etc.)

- **[bot/services/](bot/services/)** - High-level business logic
  - `UserService` - User queries and operations
  - `BroadcastService` - Mass sending with rate limiting and batching (100 users at a time)

- **[bot/managers/](bot/managers/)** - Low-level managers for external services
  - `DatabaseManager` - Tortoise ORM initialization/closing
  - `RedisManager` - Redis connection and operations
  - `i18nManager` - Locale management

- **[bot/routes/](bot/routes/)** - FastAPI webhook endpoint

- **[bot/locales/](bot/locales/)** - Fluent `.ftl` translation files (ru, en)

- **[bot/utils/](bot/utils/)** - Helper utilities
  - `Template` - Unified interface for sending/editing messages with photos, media groups, documents
  - Other helpers as needed

## Configuration & Environment

The `.env` file controls everything (required before running):

- **COMPOSE_ENV** - `dev` or `prod` (determines which compose files are loaded)
- **BOT_TOKEN** - Telegram bot token from @BotFather
- **WEBHOOK_URL** - Public HTTPS URL where bot receives updates
- **POSTGRESQL** - Connection details (PG_HOST, PG_USER, PG_PASSWORD, PG_DATABASE)
- **REDIS** - Connection details (REDIS_HOST, REDIS_PASSWORD, REDIS_DATABASE)
- **LOGGING_ENABLED** - Send errors to Telegram if True
- **ADMIN_IDS** - List of admin user IDs (format: `[123,456]`)
- **DEFAULT_LANGUAGE** - Default language code (ru, en)

## Common Development Tasks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Er1one/telegram-bot-template](https://github.com/Er1one/telegram-bot-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
