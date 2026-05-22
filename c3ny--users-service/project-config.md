---
trigger: always_on
description: This microservice follows **Hexagonal Architecture** (Ports and Adapters), which promotes:
---


# Users Service - Hexagonal Architecture Overview

## Architecture Pattern

This microservice follows **Hexagonal Architecture** (Ports and Adapters), which promotes:

- Clear separation of concerns
- Business logic independence from frameworks and external dependencies
- Testability and maintainability
- Dependency inversion principle

## Project Structure

### Core Layers

#### 1. Application Core (`src/application/core/`)

The **innermost layer** containing pure business logic with NO external dependencies.

**Domain Entities** (`src/application/core/domain/`)

- Pure TypeScript classes/interfaces representing business concepts
- Examples: [user.entity.ts](mdc:src/application/core/domain/user.entity.ts), [donor.entity.ts](mdc:src/application/core/domain/donor.entity.ts), [company.entity.ts](mdc:src/application/core/domain/company.entity.ts)
- Must NOT import from adapters, NestJS, or any framework
- Should only use native TypeScript types and other domain entities

**Services** (`src/application/core/service/`)

- Orchestrates use cases
- Example: [users.service.ts](mdc:src/application/core/service/users.service.ts)
- Depends only on use case interfaces (ports/in)
- Must NOT contain business logic (delegate to use cases)

**Errors** (`src/application/core/errors/`)

- Domain-specific error definitions
- Example: [errors.enum.ts](mdc:src/application/core/errors/errors.enum.ts)

#### 2. Application Ports (`src/application/ports/`)

Define **interfaces** for communication between layers.

**Input Ports** (`src/application/ports/in/`)

- Define use case interfaces
- Examples: [createUser.useCase.ts](mdc:src/application/ports/in/user/createUser.useCase.ts), [changePassword.useCase.ts](mdc:src/application/ports/in/user/changePassword.useCase.ts), [updateUserAvatar.useCase.ts](mdc:src/application/ports/in/user/updateUserAvatar.useCase.ts)
- Implement the `UseCase<T, R>` interface from [useCase.types.ts](mdc:src/types/useCase.types.ts)
- Return `Result<T>` type for consistent error handling
- Can inject output ports (repositories) via dependency injection

**Output Ports** (`src/application/ports/out/`)

- Define repository/external service interfaces
- Examples: [users-repository.port.ts](mdc:src/application/ports/out/users-repository.port.ts), [donor-repository.port.ts](mdc:src/application/ports/out/donor-repository.port.ts)
- Abstractions that adapters must implement
- Use domain entities, NOT framework-specific types

#### 3. Adapters (`src/adapters/`)

Connect the application to the outside world.

**Input Adapters** (`src/adapters/in/`)

- HTTP controllers, event listeners, CLI handlers
- Example: [user.controller.ts](mdc:src/adapters/in/user.controller.ts)
- Use NestJS decorators (`@Controller`, `@Post`, etc.)
- Translate HTTP requests to use case calls
- Depend on services from `application/core/service/`

**Output Adapters** (`src/adapters/out/`)

- Repository implementations, external API clients
- Examples: [users.repository.ts](mdc:src/adapters/out/users.repository.ts), [donor.repository.ts](mdc:src/adapters/out/donor.repository.ts)
- Implement port interfaces from `application/ports/out/`
- Handle framework-specific logic (TypeORM, HTTP clients, etc.)

**Adapter Domain Models** (`src/adapters/out/domain/`)

- Framework-specific entity mappings (e.g., TypeORM entities)
- Examples: [user.entity.ts](mdc:src/adapters/out/domain/user.entity.ts), [donor.entity.ts](mdc:src/adapters/out/domain/donor.entity.ts)
- Map between domain entities and persistence models

**Mappers** (`src/adapters/out/mappers/`)

- Transform between domain entities and persistence entities
- Examples: [user.mapper.ts](mdc:src/adapters/out/mappers/user.mapper.ts), [donor.mapper.ts](mdc:src/adapters/out/mappers/donor.mapper.ts)

### Shared Types (`src/types/`)

- Common type definitions used across layers
- [user.types.ts](mdc:src/types/user.types.ts) - User-related types
- [result.types.ts](mdc:src/types/result.types.ts) - Result pattern for error handling
- [useCase.types.ts](mdc:src/types/useCase.types.ts) - Use case interface

### Constants (`src/constants/`)

- Injection tokens and application constants
- Examples: `USERS_REPOSITORY`, `DONOR_REPOSITORY`, `COMPANY_REPOSITORY` tokens for DI

## Dependency Rules

### ✅ Allowed Dependencies

```
Adapters/In → Application/Core/Service → Application/Ports/In → Application/Ports/Out
                                                                          ↑
Adapters/Out ─────────────────────────────────────────────────────────────┘
```

1. **Controllers** (adapters/in) → Services (application/core/service)
2. **Services** → Use Cases (application/ports/in)
3. **Use Cases** → Repository Ports (application/ports/out)
4. **Repositories** (adapters/out) → Repository Ports (implements interface)
5. **Everyone** → Domain Entities (application/core/domain)
6. **Everyone** → Shared Types (src/types)

### ❌ Forbidden Dependencies

1. **Domain entities** MUST NOT import from:
   - NestJS (`@nestjs/*`)
   - TypeORM or any ORM
   - Adapters layer
   - Any framework-specific code

2. **Use Cases** (ports/in) MUST NOT import from:
   - Adapters layer
   - Services (circular dependency)
   - Framework implementations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c3ny/users-service](https://github.com/c3ny/users-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
