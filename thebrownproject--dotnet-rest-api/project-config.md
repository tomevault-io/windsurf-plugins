---
trigger: always_on
description: This file provides essential project context and development guidelines for the ASP.NET Core veterinary practice management API. This project integrates modern minimal API patterns with Supabase backend services.
---

# Veterinary API Project Guide for Claude Code

This file provides essential project context and development guidelines for the ASP.NET Core veterinary practice management API. This project integrates modern minimal API patterns with Supabase backend services.

> **📋 Documentation Principle**: Keep this file concise and human-readable (following Anthropic's best practices). Aim for 200-400 lines maximum. Focus on essential information Claude needs to work effectively with your project.

## 🎯 Project Context

**Purpose**: ASP.NET Core 9.0 minimal API for veterinary practice management with Supabase backend integration
**Target Users**: Veterinary professionals, practice management staff, and API consumers  
**Tech Stack**: C# .NET 9.0 + ASP.NET Core Minimal API + Supabase PostgreSQL
**Key Features**: Vet contact CRUD operations, clinic management, real-time data sync, interactive API documentation

## 🛠️ Essential Commands

### Development
- `dotnet run` - Start development server with hot reload
- `dotnet build --configuration Release` - Build for production
- `dotnet watch run` - Start with file watching and hot reload

### Testing  
- `dotnet test` - Run all tests (when implemented)
- `dotnet test --watch` - Run tests in watch mode
- `dotnet test --collect:"XPlat Code Coverage"` - Generate test coverage

### Code Quality
- `dotnet format` - Format code according to .editorconfig
- `dotnet build` - Check for compilation errors and warnings
- Built-in Roslyn analyzers provide static analysis

**IMPORTANT**: Always run `dotnet build` and `dotnet format` before committing.

## 📦 Technology Stack

**Runtime**: .NET 9.0 SDK
**Framework**: ASP.NET Core 9.0 Minimal API
**Database**: Supabase (PostgreSQL-based) with real-time capabilities
**Key Dependencies**: Supabase (1.1.1), Newtonsoft.Json (13.0.3), Swashbuckle.AspNetCore (9.0.3)
**Dev Tools**: .NET CLI, Visual Studio Code, Swagger UI, EditorConfig, Roslyn Analyzers

## 📁 Project Structure

```
min-api-project/
├── Controllers/            # API controllers (VetController.cs)
├── Models/                # Data models and DTOs  
├── Contracts/             # API request/response contracts
├── Validation/            # Input validation logic
├── Pages/                 # Razor Pages for UI (future)
├── wwwroot/              # Static web assets
├── Properties/           # Launch settings and configuration
├── Program.cs            # Application entry point
├── appsettings.json      # Application configuration
└── CLAUDE.md             # Project documentation
```

**File Organization**: Feature-based controllers with clean separation of models, contracts, and validation
**New Files**: Controllers in `Controllers/`, DTOs in `Models/`, API contracts in `Contracts/`, services in new `Services/` directory

## 🎨 Code Style & Conventions

### General Guidelines
- **File Size**: Maximum 500 lines - split larger files
- **Function Size**: Keep under 50 lines with single responsibility
- **Line Length**: 100 characters maximum
- **Imports**: Using directives at top of file, organized by framework then project

### Naming Conventions
- **Files**: PascalCase matching class names (VetController.cs)
- **Variables**: camelCase for local variables and parameters
- **Functions**: PascalCase for public methods, camelCase for private
- **Classes**: PascalCase with descriptive nouns

### Project-Specific Rules
- Follow Microsoft C# coding conventions
- One class per file, namespace matches folder structure
- Use async/await for I/O operations
- Constructor injection with interface abstractions
- Thin controllers with business logic in services

## 🧪 Testing Strategy

**Framework**: xUnit (planned) with Moq for mocking
**Coverage Target**: 80%+ with focus on critical paths
**Test Structure**: Separate test projects following .NET naming conventions

### Testing Requirements (To Implement)
- **All new features** must include unit tests
- **Bug fixes** require regression tests
- **API endpoints** need integration tests with TestServer
- **Controllers** need unit tests with mocked dependencies

**IMPORTANT**: Run `dotnet test` before every commit once tests are implemented.

## 🏗️ Architecture Principles

### Core Philosophy
- **KISS**: Choose simple solutions over complex ones
- **YAGNI**: Implement only what's needed now
- **Single Responsibility**: Each module has one clear purpose
- **Fail Fast**: Validate early, raise exceptions immediately

### Error Handling
- Use specific exceptions, not broad catches
- Log structured information with appropriate levels
- Validate all user input at system boundaries
- Clean up resources properly (use context managers)

### Security
- **Never commit secrets** - use environment variables
- **Validate input** at all boundaries
- **Use parameterized queries** - never concatenate user input
- **Keep dependencies updated** and monitor vulnerabilities

## 🔄 Git Workflow

**Branch Strategy**: `main` ← PR ← `feature/`, `fix/`, `docs/`
**Commit Format**: `type(scope): description` (e.g., `feat(auth): add login validation`)
**Daily Flow**: checkout main → pull → create branch → develop → test → PR


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thebrownproject/dotnet-rest-api](https://github.com/thebrownproject/dotnet-rest-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
