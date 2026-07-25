---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ABP Framework-based microservices template using .NET Aspire for orchestration. The solution demonstrates a production-grade microservices architecture with four core services (Administration, Identity, Projects, SaaS), a YARP-based API gateway, OpenIddict authentication server, and Blazor WebAssembly frontend.

**Current ABP Version:** 10.0
**Target Framework:** .NET 10.0
**.NET Aspire Version:** 9.5.2

## Quick Start with Makefile

A comprehensive Makefile is available at the repository root for common development tasks:

```bash
# First-time setup
make install          # Restore tools and install git hooks

# Daily workflow
make build            # Build the solution
make test             # Run all tests
make format           # Format code with CSharpier
make fix              # Auto-fix analyzers + style + format
make run              # Run with .NET Aspire orchestration

# Database operations
make migrate          # Run database migrations
make reset-db         # Reset databases (dev only - DESTRUCTIVE)

# Individual services
make run-gateway      # Run API Gateway only
make run-auth         # Run AuthServer only
make run-admin        # Run Administration service only

# Code quality
make check-format     # Check formatting without changes
make warnings         # Show build warnings/errors
make analyzers        # Show analyzer diagnostics

# Help
make help             # Show all available targets
make dev-help         # Show quick start guide
```

All commands below can also be run using the Makefile shortcuts shown above.

## Development Commands

### Building the Solution

```bash
# Build entire solution from src/ directory
dotnet build Tasky.sln

# Build specific service
dotnet build services/administration/host/Tasky.Administration.HttpApi.Host/Tasky.Administration.HttpApi.Host.csproj

# Clean build artifacts
clean.bat  # or manually: dotnet clean
```

### Running the Application

```bash
# Run with .NET Aspire orchestration (recommended)
cd apps/Tasky.AppHost
dotnet run

# This starts:
# - PostgreSQL (with 4 databases)
# - RabbitMQ (with management UI)
# - Redis (with Redis Commander)
# - Seq (log aggregation)
# - DbMigrator (runs migrations)
# - All microservices (Administration, Identity, Projects, SaaS)
# - API Gateway
# - AuthServer
# - Blazor WebApp

# Access Aspire Dashboard at: http://localhost:15888 (check console output for exact URL)
```

### Running Individual Services

```bash
# Run a single microservice (without Aspire)
cd services/administration/host/Tasky.Administration.HttpApi.Host
dotnet run

# Note: Requires manual infrastructure setup (PostgreSQL, Redis, RabbitMQ)
# Connection strings in appsettings.json must point to running infrastructure
```

### Testing

```bash
# Run all tests in solution
dotnet test Tasky.sln

# Run tests for specific service
dotnet test services/administration/test/Tasky.Administration.Application.Tests/Tasky.Administration.Application.Tests.csproj

# Run single test class
dotnet test --filter "FullyQualifiedName~SampleAppService_Tests"

# Run tests with detailed output
dotnet test --logger "console;verbosity=detailed"
```

### Database Migrations

```bash
# Run migrations manually (normally handled by Aspire startup)
cd shared/Tasky.DbMigrator
dotnet run

# Add new migration to a service (example: Administration)
cd services/administration/src/Tasky.Administration.EntityFrameworkCore
dotnet ef migrations add MigrationName
```

### Database Reset (Development Only)

**Recommended: Using Aspire Dashboard**

When running the application with Aspire (`cd apps/Tasky.AppHost && dotnet run`):

1. Open the Aspire Dashboard (typically http://localhost:15888)
2. Navigate to the "Tasky-DbMigrator" resource
3. Click the **"Reset Databases"** button in the resource commands section
4. The command will drop and recreate all databases with fresh migrations

This custom command is configured in `apps/Tasky.AppHost/Program.cs` and provides visual feedback in the dashboard.

**Alternative: Manual Script Execution**

```bash
# On Windows (PowerShell)
.\reset-databases.ps1

# On Linux/macOS
chmod +x reset-databases.sh
./reset-databases.sh

# Manual reset (set environment variable directly)
cd shared/Tasky.DbMigrator
$env:RESET_DATABASES="true"  # PowerShell
# or
export RESET_DATABASES=true  # Bash
dotnet run
```

**What the reset does:**
- Drops all four databases (Administration, Identity, Projects, SaaS)
- Recreates databases with fresh schema from migrations
- Seeds permissions, features, and settings
- Creates default admin user: `admin` / `1q2w3E*`
- Seeds OpenIddict OAuth2 clients

**WARNING:** This permanently deletes ALL data in all databases!

### Code Formatting and Auto-Fixes

This project uses automated tools to format code and fix analyzer issues:

```bash
# Format entire solution with CSharpier
dotnet csharpier .

# Auto-fix analyzer diagnostics (Roslynator, SonarAnalyzer, etc.)
dotnet format analyzers src/Tasky.sln

# Auto-fix code style issues (IDE* rules)
dotnet format style src/Tasky.sln

# Run all auto-fixes and formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antosubash/abp-microservice](https://github.com/antosubash/abp-microservice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
