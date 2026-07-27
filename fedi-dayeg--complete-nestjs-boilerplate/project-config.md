---
trigger: always_on
description: This document provides instructions for GitHub Copilot to generate code that follows the patterns, conventions, and architecture of this project.
---

# GitHub Copilot Instructions for ACK NestJS Boilerplate

This document provides instructions for GitHub Copilot to generate code that follows the patterns, conventions, and architecture of this project.

> **Copilot Behavior Note**
> - Every suggestion or implementation **must consider best practices** (security, maintainability, scalability, readability).
> - If an approach is chosen for **performance** reasons or to **match an existing implementation**, **it must be explicitly noted** — e.g., with a comment like `// NOTE: this approach is chosen for performance` or `// NOTE: aligned with existing pattern in codebase`.
> - If an example is needed to clarify a suggestion, **provide the code example immediately** — do not defer or ask for confirmation first.

## Project Overview

Complete NestJS Boilerplate (v8.2.0+) is a comprehensive authentication and authorization service built with NestJS v11.x. It uses:
- **Prisma ORM** for database operations (MongoDB with replication set)
- **PNPM** as the package manager
- **Repository Design Pattern** for data access layer
- **Modular Architecture** with clear separation of concerns
- **SOLID Principles** throughout the codebase
- **Redis** for caching and session management
- **BullMQ** for queue processing
- **ES256/ES512** JWT algorithms for authentication
- **RedisSessionStore** for session handling
- **class-validator** and **class-transformer** for DTO validation and transformation
- **Swagger** for API documentation
- **i18n** for internationalization with nested JSON structure
- **BullMQ** for background job processing
- **TypeScript** with strict path aliases

## Architecture Patterns

### Module Structure

Every feature module follows this structure:
```
module/
├── bases/              # Abstract base classes for shared functionality
├── constants/          # Static values and configuration
├── controllers/        # API endpoint handlers
├── decorators/         # Custom metadata decorators
├── docs/              # Swagger/OpenAPI documentation decorators
├── dtos/              # Data Transfer Objects with validation
├── entities/          # Database entity types
├── enums/             # Type-safe enumerations
├── exceptions/        # Custom error classes
├── factories/         # Object creation patterns
├── filters/           # Exception/validation filters
├── guards/            # Authorization and access control
├── interceptors/      # Request/response transformation
├── interfaces/        # TypeScript contracts
├── middlewares/       # Request preprocessing
├── pipes/             # Data transformation and validation
├── processors/        # Background job handlers (BullMQ)
├── repositories/      # Data access layer (Prisma)
├── services/          # Business logic
├── templates/         # Email/document templates
├── utils/             # Helper utilities
└── validations/       # Custom validators
```

### Repository Pattern with Prisma

Always use repository pattern for database operations. The `DatabaseService` extends `PrismaClient`, so inject it directly without `@Inject`:

```typescript
// Repository — no interface needed, inject directly as class
@Injectable()
export class UserRepository {
    constructor(private readonly databaseService: DatabaseService) {}

    async findById(id: string): Promise<User | null> {
        return this.databaseService.user.findFirst({
            where: { id }
        });
    }
}

// Service Interface — services always have an interface
export interface IUserService {
    findById(id: string): Promise<User | null>;
    // ... other methods
}

// Service Implementation — injects repository directly as class
export class UserService implements IUserService {
    constructor(
        private readonly userRepository: UserRepository
    ) {}
}
```

### Path Aliases

Always use TypeScript path aliases defined in `tsconfig.json`:
- `@app/*` → `src/app/*`
- `@common/*` → `src/common/*`
- `@config` → `src/configs/index.ts`
- `@configs/*` → `src/configs/*`
- `@modules/*` → `src/modules/*`
- `@routes/*` → `src/router/routes/*`
- `@router` → `src/router/router.module.ts`
- `@migration/*` → `src/migration/*`
- `@test/*` → `test/*`
- `@generated/*` → `generated/*`
- `@prisma/client` → `generated/prisma-client`

Example:
```typescript
import { DatabaseModule } from '@common/database/database.module';
import { IConfigAuth } from '@configs/auth.config';
import { UserModule } from '@modules/user/user.module';
```

## Authentication & Authorization

### Decorator Order

When using multiple protection decorators, apply them in this **EXACT** order (top to bottom):

```typescript
@ExampleDoc()                              // Documentation (always first)
@TermPolicyAcceptanceProtected(...)       // Terms acceptance verification
@PolicyAbilityProtected({...})            // CASL policy-based permissions
@RoleProtected(...)                       // Role-based access control
@ActivityLog(...)                         // Activity logging
@UserProtected()                          // User authentication check
@AuthJwtAccessProtected()                 // JWT access token validation
@FeatureFlagProtected(...)                // Feature flag check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fedi-dayeg/complete-nestjs-boilerplate](https://github.com/fedi-dayeg/complete-nestjs-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
