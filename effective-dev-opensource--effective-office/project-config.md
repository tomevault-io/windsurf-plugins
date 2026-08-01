---
trigger: always_on
description: Effective Office is a multi-module Kotlin application aimed at automating office processes and providing statistics for employees. The project follows a client-server architecture with a Spring Boot backend, multiple client applications including an iOS tablet app, and Docker-based containerization for deployment.
---

# Project Guidelines for Junie

## Project Overview
Effective Office is a multi-module Kotlin application aimed at automating office processes and providing statistics for employees. The project follows a client-server architecture with a Spring Boot backend, multiple client applications including an iOS tablet app, and Docker-based containerization for deployment.

## Project Structure
```
effective-office/
├── backend/           # Server-side Spring Boot application with PostgreSQL
├── clients/           # Client applications
├── iosApp/            # iOS tablet application
├── deploy/            # Deployment configurations (dev and prod environments)
├── scripts/           # Utility scripts and git hooks
├── build-logic/       # Build configuration
├── docs/              # Project documentation
└── media/             # Media assets
```

## Running Tests
Tests can be run using Gradle:
```
./gradlew test
```

For specific modules:
```
./gradlew :backend:app:test
./gradlew :clients:android:test
```

## Build Guidelines
Before submitting changes, Junie should build the project to ensure it compiles correctly:
```
./gradlew build
```

## Code Style Guidelines
1. Follow Kotlin coding conventions for all development
2. Use consistent naming patterns across the codebase
3. Document public APIs and complex logic
4. Ensure no secrets or sensitive information is included in the code
5. Maintain the multi-module structure of the project

## Security Guidelines
1. Never commit sensitive information like API keys or credentials
2. Use environment variables for configuration as shown in the .env.example files
3. Be aware of the pre-commit hooks that scan for potential secrets using Gitleaks

## Feature Development Guidelines
### Modular Architecture
1. Every feature should be implemented as a separate feature module
2. Feature modules should be independent and self-contained
3. Feature modules should only depend on core modules, not on other feature modules
4. New features should be added by creating a new module in the appropriate directory:
   ```
   effective-office/
   ├── backend/
   │   ├── features/
   │   │   ├── feature-a/     # Feature module A
   │   │   ├── feature-b/     # Feature module B
   │   │   └── new-feature/   # New feature module
   ```

### Core Entities
1. Core entities must be placed in appropriate core modules
2. Domain models should be in the `domain` module
3. Data models should be in the `data` module
4. UI components should be in the `ui` module

### Clean Architecture
The project follows Clean Architecture principles:
1. **Domain Layer**: Contains business logic and entities
   - Independent of frameworks and UI
   - Contains use cases, entities, and repository interfaces
2. **Data Layer**: Implements repository interfaces from the domain layer
   - Contains repository implementations, data sources, and mappers
3. **Presentation Layer**: Contains UI components and view models
   - Depends on the domain layer, not on the data layer
4. **Dependencies flow from outer layers to inner layers**:
   ```
   UI/Presentation → Domain ← Data
   ```

### SOLID Principles
1. **Single Responsibility Principle**: Each class should have only one reason to change
2. **Open/Closed Principle**: Software entities should be open for extension but closed for modification
3. **Liskov Substitution Principle**: Objects of a superclass should be replaceable with objects of subclasses without affecting correctness
4. **Interface Segregation Principle**: Many client-specific interfaces are better than one general-purpose interface
5. **Dependency Inversion Principle**: Depend on abstractions, not on concretions

### DRY and KISS Patterns
1. **DRY (Don't Repeat Yourself)**:
   - Avoid code duplication
   - Extract common functionality into reusable components
   - Use shared modules for code that's used across multiple features
2. **KISS (Keep It Simple, Stupid)**:
   - Prefer simple solutions over complex ones
   - Write code that is easy to understand and maintain
   - Avoid premature optimization

## Client Application Development Guidelines
### Navigation Framework
1. **Compose Navigation**:
   - Use Jetpack Navigation Component for Android applications
   - Implement type-safe navigation with sealed classes for destinations
   - Use NavHost for managing navigation between composables

2. **KMP Navigation**:
   - Use Decompose for shared navigation logic across platforms
   - Implement platform-specific navigation adapters
   - Maintain a consistent navigation state model across platforms

### UI Implementation
1. **Compose Multiplatform**:
   - Use Compose Multiplatform for KMP applications for shared UI components across platforms
   - Implement a design system with shared tokens (colors, typography, spacing)
   - Create platform-specific UI adaptations when necessary
   - Use expect/actual declarations for platform-specific UI implementations

2. **Responsive Design**:
   - Implement responsive layouts that adapt to different screen sizes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [effective-dev-opensource/Effective-Office](https://github.com/effective-dev-opensource/Effective-Office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
