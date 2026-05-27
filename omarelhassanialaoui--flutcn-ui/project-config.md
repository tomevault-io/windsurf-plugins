---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flutcn UI is a CLI tool for Flutter developers that automates UI theme setup and widget management. It fetches pre-built components from a remote registry (https://flutcnui.netlify.app) and integrates them into Flutter projects with customizable themes.

## Common Commands

### Development & Testing

```bash
# Install dependencies
dart pub get

# Install CLI globally from source
dart pub global activate --source path .

# Run the CLI (when installed globally)
flutcn_ui init
flutcn_ui add <widget-name>
flutcn_ui list
flutcn_ui remove <widget-name>
flutcn_ui update <widget-name>

# Run the CLI directly from source (without global install)
dart run bin/flutcn_ui.dart init
dart run bin/flutcn_ui.dart add button
dart run bin/flutcn_ui.dart list
dart run bin/flutcn_ui.dart remove button
dart run bin/flutcn_ui.dart update button
dart run bin/flutcn_ui.dart update --all

# Linting
dart analyze

# Run tests (47 unit tests)
dart test

# Build runner (if code generation is added)
dart run build_runner build --delete-conflicting-outputs
```

### Testing CLI Commands Locally

The CLI must be run from within a Flutter project directory (where `pubspec.yaml` exists). For testing:

```bash
cd example
dart run ../bin/flutcn_ui.dart init --default
dart run ../bin/flutcn_ui.dart add button
dart run ../bin/flutcn_ui.dart remove button
dart run ../bin/flutcn_ui.dart update button
dart run ../bin/flutcn_ui.dart update --all
```

## Architecture

### Clean Architecture Layers

The codebase follows Clean Architecture principles with clear separation:

**1. Domain Layer** (`lib/src/domain/`)
- **Entities**: Pure business objects (e.g., `WidgetEntity`, `InitConfigEntity`)
- **Repositories**: Abstract interfaces defining contracts
- **Use Cases**: Business logic encapsulated in single-responsibility classes
- Dependencies: None (most inner layer)

**2. Data Layer** (`lib/src/data/`)
- **Models**: Data transfer objects that extend entities with JSON serialization
- **Repositories**: Concrete implementations of domain repository interfaces
- **Interfaces**: Abstract data source contracts (e.g., `CommandInterface`)
- **Services**: Implementation of interfaces (e.g., `CommandInterfaceImpl`, `ApiService`)
- Dependencies: Only on domain layer

**3. Presentation Layer** (`bin/`)
- **Commands**: CLI command implementations using `args` package
- **Injection Container**: Dependency injection setup using GetIt
- Dependencies: On both domain and data layers (outer layer)

### Key Patterns

**Dependency Injection with GetIt**
- Registration happens in `bin/injection_container.dart`
- Use `sl<Type>()` to retrieve dependencies in commands
- Layer order: Use Cases → Repository → Data Sources → Services

**Functional Error Handling**
- Uses `dartz` package for `Either<Failure, Success>` pattern
- Left side: Failure (error cases)
- Right side: Success value
- Repositories return `Either<Failure, T>`, use cases pass through `Either`
- Commands unwrap `Either` with `.fold()` inside spinner actions

**Entity-Model Pattern**
- Entities: Pure domain objects in `domain/entities/`
- Models: Data layer objects with `toModel()` and `fromJSON()` in `data/models/`
- Models extend entities and add serialization capabilities

### Data Flow Example

```
CLI Command (bin/commands/)
    ↓
Use Case (domain/usecases/)
    ↓
Repository Interface (domain/repository/)
    ↓
Repository Implementation (data/repository/)
    ↓
Data Interface (data/interfaces/)
    ↓
Data Service Implementation (data/services/)
    ↓
API Service (core/services/)
```

## Core Components

### API Service
- Abstract `ApiService` interface in `lib/src/core/services/api_service.dart`
- HTTP implementation in `lib/src/data/services/api_service.dart`
- Base URLs configured in `lib/src/core/constants/api_constants.dart`
- Fetches widgets and themes from `https://flutcnui.netlify.app/registry/`

### Configuration File
- Generated at project root: `flutcn.config.json`
- Stores: widgets path, theme path, style, base color
- Must exist before running `add`, `list`, `remove`, or `update` commands

### Spinner Helper
- Located in `lib/src/core/utils/spinners.dart`
- Provides visual feedback for long-running operations
- Pattern: `_spinnerHelper.runWithSpinner(message, onSuccess, onError, action)`

## Adding New Commands

1. Create command class in `bin/commands/<command_name>.dart` extending `Command`
2. Implement required fields: `name`, `description`
3. Override `run()` method
4. Register in `bin/flutcn_ui.dart` with `runner.addCommand(YourCommand())`
5. Create corresponding use case in `lib/src/domain/usecases/`
6. Add repository method if needed
7. Register use case in `bin/injection_container.dart`

## Widget Registry Integration

Widgets are fetched from the remote registry API:
- Registry base URL: `https://flutcnui.netlify.app/registry/`
- Widget list endpoint: `/widgets`
- Individual widget files: returned in widget metadata
- Theme files: `/colorScheme/{style}/{baseColor}` and `/theme/{style}`

## File Structure Conventions

When CLI runs `init`:
- Creates `flutcn.config.json` in project root

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OmarElhassaniAlaoui/flutcn_ui](https://github.com/OmarElhassaniAlaoui/flutcn_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
