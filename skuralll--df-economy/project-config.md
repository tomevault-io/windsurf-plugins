---
trigger: always_on
description: Economic system for Minecraft Dragonfly server
---

# DF Economy Project

Economic system for Minecraft Dragonfly server

## About Dragonfly
- Asynchronous Minecraft: Bedrock Edition server written in Go
- Extensibility-focused design, used as a library
- Struct-based command system with reflection-based argument parsing
- Client-side command integration (auto-completion, validation)

## Command System Basics
- Implement `cmd.Runnable` interface
- Struct fields become command parameters
- Use `cmd.SubCommand` to define subcommands
- Use `cmd.Optional[T]` for optional arguments
- Commands registered via `cmd.Register()` with `cmd.New()`
- Error handling through `cmd.Output` and direct player messaging

## Project Structure
```
df-economy/
├── dragonfly/commands/     # Command implementations
│   ├── base.go            # BaseCommand with shared utilities
│   ├── register.go        # Command registration
│   ├── economy.go         # Main help command
│   ├── balance.go         # Balance display (supports optional target)
│   ├── set.go             # Balance setting (configurable command)
│   ├── top.go             # Rankings with pagination
│   └── pay.go             # Money transfer between players
├── economy/               # Domain and business logic
│   ├── domain.go          # EconomyEntry domain model
│   ├── config/            # Configuration management
│   │   └── config.go      # Economy configuration struct with multi-RDBMS and command control
│   └── service/           # Business logic layer
│       ├── service.go     # EconomyService with validation
│       └── errors.go      # Service-specific error types
├── internal/db/           # Database abstraction
│   ├── db.go              # DB interface
│   ├── factory.go         # Database factory for multi-RDBMS support
│   ├── gorm.go            # GORM implementation with multi-RDBMS support
│   ├── model.go           # Account database model
│   └── errors.go          # Database-specific error types
├── cmd/demo/              # Demo server implementation
│   └── main.go            # Server setup with economy integration
└── config.toml            # Dragonfly server configuration
```

## Architecture Design
- **Domain Layer**: Clean `EconomyEntry` struct representing core business objects
- **Service Layer**: `EconomyService` handles business logic, validation, and configuration
- **Database Layer**: Interface-based design with GORM ORM implementation supporting multiple RDBMSs
- **Command Layer**: Dragonfly commands with async execution
- **Error Handling**: Custom error types with user-friendly messages, layered by responsibility
- **Configuration**: Configurable database type, DSN, default balance, and command availability settings
- **Factory Pattern**: Database factory for multi-RDBMS support (SQLite, MySQL, PostgreSQL)
- **Async Processing**: Non-blocking command execution with timeout handling

## Command Implementation Patterns
- **BaseCommand**: Shared utilities for player validation, async execution, UUID resolution
- **Async Execution**: All database operations run asynchronously with 5-second timeout
- **Error Messaging**: Consistent error handling with colored chat messages
- **Immediate Feedback**: Commands provide instant feedback before async processing

## Database Operations
- **Multi-RDBMS Support**: SQLite, MySQL, and PostgreSQL support via GORM ORM
- **Factory Pattern**: Database factory (`NewDB`) creates appropriate dialector based on configuration
- **Account Model**: Database schema with UUID, Name, Balance fields using GORM tags
- **Balance Management**: Get, Set, Transfer operations with atomic transactions
- **User Registration**: Automatic user registration on first join with configurable default balance
- **Leaderboards**: Paginated top balance queries with efficient ordering
- **UUID Resolution**: Name-to-UUID mapping for player operations
- **Schema Migration**: Automatic database schema migration on startup
- **CGO-Free**: All database drivers are pure Go implementations, no CGO required

## Domain Model
- **EconomyEntry**: Core domain struct with UUID, Name, Balance fields
- **Account**: Database model with GORM tags and automatic timestamps
- **Balance Field**: Renamed from "Money" to "Balance" for clarity
- **Configuration**: Multi-RDBMS configuration with database type, DSN, and default balance

## Configuration System
- **DBType**: Database type selection ("sqlite", "mysql", "postgres")
- **DBDSN**: Database connection string or path
- **DefaultBalance**: Initial balance for new users
- **EnableSetCmd**: Enable/disable the `/economy set` command (default: false)
- **String-Based Configuration**: Practical string-based settings for external library integration
- **Multi-RDBMS Support**: Configurable database backend selection
- **Command Control**: Selective command availability for enhanced security

## Security & Validation
- **Input Validation**: Amount validation (positive, minimum values)
- **Self-Transfer Prevention**: Cannot pay yourself
- **Command Access Control**: Set command disabled by default for security
- **Configurable Permissions**: Server admins control command availability
- **Timeout Handling**: All operations have 5-second timeout
- **Error Logging**: Internal errors logged with context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skuralll) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
