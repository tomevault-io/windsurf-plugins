---
trigger: always_on
description: Notifuse is a modern, self-hosted email marketing platform built with a clean architecture approach. The application follows a microservices-inspired design with clear separation between frontend and backend components.
---

# Notifuse Tech Stack Documentation

## Overview

Notifuse is a modern, self-hosted email marketing platform built with a clean architecture approach. The application follows a microservices-inspired design with clear separation between frontend and backend components.

## 🏗️ Architecture

The application follows **Clean Architecture** principles with distinct layers:

- **Domain Layer**: Core business logic and entities
- **Service Layer**: Business logic implementation
- **Repository Layer**: Data access and storage
- **HTTP Layer**: API handlers and middleware
- **Frontend Layer**: Multiple React-based user interfaces

## 🔧 Backend Tech Stack

### Core Framework & Language

- **Language**: Go 1.25.x
- **HTTP Framework**: Standard library `http.ServeMux` (no external web framework)
- **Architecture**: Clean Architecture with dependency injection

### Database & Storage

- **Primary Database**: PostgreSQL 17
- **Query Builder**: Squirrel for type-safe SQL queries
- **Migrations**: Custom migration system with version-based schema management
- **Connection Pooling**: Built-in database/sql with OpenCensus integration

### Database Migration System

Notifuse uses a custom migration system that manages database schema changes across both the system database and individual workspace databases. The migration system is designed to handle schema evolution safely and consistently.

#### Version Format

- **Tag Format**: `vMAJOR.minor` (e.g., `v6.5`, `v4.0`)
- **Major Version**: Incremented when database schemas change
- **Minor Version**: Incremented for non-schema changes (features, bug fixes)
- **Code Version**: Stored in `config/config.go` as `VERSION = "6.5"`

#### Migration Types

The system supports two types of migrations:

1. **System Migrations**: Changes to the main system database schema

   - User management tables
   - Workspace management tables
   - System configuration tables
   - Global settings and permissions

2. **Workspace Migrations**: Changes to individual workspace database schemas
   - Contact management tables
   - Template and broadcast tables
   - Message history tables
   - Workspace-specific settings

#### Migration Structure

Each migration implements the `MajorMigrationInterface`:

```go
type MajorMigrationInterface interface {
    GetMajorVersion() float64                    // Returns major version (e.g., 6.0)
    HasSystemUpdate() bool                       // Indicates system database changes
    HasWorkspaceUpdate() bool                    // Indicates workspace database changes
    UpdateSystem(ctx, config, db) error          // Executes system migrations
    UpdateWorkspace(ctx, config, workspace, db) error // Executes workspace migrations
}
```

#### Migration Execution

1. **Version Comparison**: System compares current database version with code version
2. **Migration Selection**: Identifies migrations that need to be executed
3. **System Updates**: Executes system migrations in a transaction
4. **Workspace Updates**: For each workspace, connects to its database and executes workspace migrations
5. **Version Update**: Updates database version after successful completion

#### Migration Safety

- **Idempotent**: Migrations can be run multiple times safely using `IF NOT EXISTS` clauses
- **Transactional**: Each migration runs in a database transaction
- **Rollback**: Failed migrations are automatically rolled back
- **Backward Compatibility**: Existing data is preserved with default values

#### Example Migration

```go
// V6Migration adds permissions system
type V6Migration struct{}

func (m *V6Migration) GetMajorVersion() float64 { return 6.0 }
func (m *V6Migration) HasSystemUpdate() bool { return true }
func (m *V6Migration) HasWorkspaceUpdate() bool { return false }

func (m *V6Migration) UpdateSystem(ctx context.Context, config *config.Config, db DBExecutor) error {
    // Add permissions column to user_workspaces table
    _, err := db.ExecContext(ctx, `
        ALTER TABLE user_workspaces
        ADD COLUMN IF NOT EXISTS permissions JSONB DEFAULT '{}'::jsonb
    `)
    return err
}
```

#### Changelog Integration

- **CHANGELOG.md**: Updated with each version release
- **Version History**: Tracks all schema changes and their impact
- **Breaking Changes**: Clearly documented for upgrade planning

### Authentication & Security

- **Token System**: JWT (Platform-Agnostic Security Tokens)
- **Password Hashing**: bcrypt via golang.org/x/crypto
- **API Security**: Custom middleware for authentication and CORS

### Email & Communication

- **Email Engine**: Multiple provider support:
  - Amazon SES (AWS SDK v1.55.7)
  - SMTP (go-mail v0.7.2)
  - Mailgun, Mailjet, Postmark, SparkPost integrations
- **Template Engine**: Liquid templating (Notifuse/liquidgo)
- **MJML Support**: gomjml v0.10.0 for email rendering
- **HTML Parsing**: PuerkitoBio/goquery v1.10.3

### Observability & Monitoring

- **Logging**: Zerolog v1.33.0 (structured logging)
- **Tracing**: OpenCensus with multiple exporters:
  - Jaeger, Zipkin, Stackdriver, DataDog, AWS X-Ray
  - Prometheus metrics integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Notifuse/notifuse](https://github.com/Notifuse/notifuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
