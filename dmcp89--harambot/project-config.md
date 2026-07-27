---
trigger: always_on
description: Harambot is an interactive Yahoo Fantasy Sports Discord bot that allows users to interact with their Yahoo Fantasy leagues directly from Discord. The bot supports multiple fantasy sports (football, basketball, baseball, hockey) and provides real-time updates, statistics, and league management features.
---

# Harambot - GitHub Copilot Instructions

## Project Overview
Harambot is an interactive Yahoo Fantasy Sports Discord bot that allows users to interact with their Yahoo Fantasy leagues directly from Discord. The bot supports multiple fantasy sports (football, basketball, baseball, hockey) and provides real-time updates, statistics, and league management features.

## Technology Stack
- **Language**: Python 3.10+
- **Framework**: discord.py 2.5.2+ (Discord bot framework)
- **API Integration**: yahoo-fantasy-api, yahoo-oauth
- **Database**: Peewee ORM (supports SQLite, PostgreSQL, MySQL)
- **Configuration**: Dynaconf (settings.toml, .secrets.toml)
- **Encryption**: cryptography (Fernet) for storing sensitive tokens
- **Web Server**: aiohttp (for OAuth callbacks)
- **Async**: asyncio for concurrent operations

## Project Structure

### Core Modules
- **harambot/bot.py**: Main bot initialization, event handlers, cog loading
- **harambot/config.py**: Configuration management using Dynaconf
- **harambot/yahoo_api.py**: Yahoo Fantasy API wrapper and interaction layer
- **harambot/utils.py**: Shared utility functions

### Cogs (Discord Command Modules)
- **cogs/meta.py**: Meta commands (configuration, setup)
- **cogs/yahoo.py**: Yahoo Fantasy commands (standings, roster, stats, matchups, etc.)
- **cogs/misc.py**: Miscellaneous commands (ping, RIP)

### Database Layer
- **database/models.py**: Peewee ORM models (Guild, etc.)
- **database/fields.py**: Custom field types (EncryptedField)
- **database/migrations.py**: Database migration logic

### Services
- **services/webserver.py**: OAuth callback web server for Yahoo authentication
- **services/reports.py**: Transaction polling and reporting service

### UI Components
- **ui/modals.py**: Discord modals for interactive forms
- **ui/views.py**: Discord views for buttons and interactive components

## Coding Conventions

### Style Guidelines
- **Line Length**: 79 characters (Black formatter configuration)
- **Formatter**: Black (target Python 3.8+)
- **Linter**: flake8
- **Imports**: Group by stdlib, third-party, local (PEP 8)
- **Logging**: Use module-level loggers: `logger = logging.getLogger("discord.harambot.module_name")`

### Naming Conventions
- **Variables/Functions**: snake_case
- **Classes**: PascalCase
- **Constants**: UPPER_SNAKE_CASE
- **Discord Commands**: lowercase with underscores
- **Cogs**: End class names with "Cog" (e.g., YahooCog)

### Discord.py Patterns
- Use `app_commands` for slash commands
- Always defer responses for long-running operations: `await interaction.response.defer()`
- Use embeds for rich formatted responses
- Handle errors gracefully with try-except blocks
- Use Discord views and modals for interactive components

### Database Patterns
- All models inherit from `BaseModel`
- Use Peewee ORM query methods
- Encrypt sensitive data (tokens) using `EncryptedField`
- Check table existence before creating: `Model.table_exists()`
- Use `playhouse.db_url.connect()` for database connection

### API Integration
- Use `yahoo_api.py` wrapper for all Yahoo API calls
- Pass `guild_id` to API methods to fetch guild-specific credentials
- Cache API responses where appropriate using `cachetools`
- Handle API errors and rate limits gracefully

## Environment Variables & Configuration

### Required Environment Variables
```bash
DISCORD_TOKEN         # Discord bot token
YAHOO_KEY            # Yahoo API client ID
YAHOO_SECRET         # Yahoo API client secret
DATABASE_URL         # Database connection URL (e.g., sqlite:///harambot.db)
HARAMBOT_KEY         # Fernet encryption key (URL-safe base64-encoded 32-byte)
```

### Optional Environment Variables
```bash
LOGLEVEL             # Logging level (DEBUG, INFO, WARNING, ERROR)
```

### Configuration Files
- **config/settings.toml**: General settings
- **config/.secrets.toml**: Sensitive credentials (gitignored)

## Common Development Tasks

### Adding a New Discord Command
1. Add command to appropriate cog in `harambot/cogs/`
2. Use `@app_commands.command()` decorator
3. Add logging at command start
4. Defer response if operation takes >3 seconds
5. Handle errors with try-except
6. Return Discord embeds for formatted output

### Adding a New Database Model
1. Create model class in `database/models.py` inheriting from `BaseModel`
2. Add field definitions using Peewee field types
3. Use `EncryptedField` for sensitive data
4. Add migration logic if modifying existing tables
5. Create table in bot initialization if needed

### Working with Yahoo API
1. Use methods in `yahoo_api.py`
2. Pass `guild_id` for guild-specific credentials
3. Handle OAuth token refresh automatically
4. Cache responses using `@cached` decorator
5. Parse Yahoo API responses carefully (nested JSON structures)

### Creating Interactive UI
1. Define views in `ui/views.py` using `discord.ui.View`
2. Define modals in `ui/modals.py` using `discord.ui.Modal`
3. Add buttons using `discord.ui.Button`
4. Handle callbacks with async methods
5. Set appropriate timeouts for views

## Testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DMcP89/harambot](https://github.com/DMcP89/harambot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
