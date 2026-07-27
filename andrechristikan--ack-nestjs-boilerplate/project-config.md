---
trigger: always_on
description: This document provides instructions for GitHub Copilot to generate code that follows the patterns, conventions, and architecture of this project.
---

# GitHub Copilot Instructions for ACK NestJS Boilerplate

This document provides instructions for GitHub Copilot to generate code that follows the patterns, conventions, and architecture of this project.

> **Copilot Behavior Note**
> - Always read relevant source files before suggesting changes — never assume code structure.
> - Every suggestion or implementation **must consider best practices** (security, maintainability, scalability, readability).
> - If an approach is chosen for **performance** reasons or to **match an existing implementation**, **it must be explicitly noted** — mark it with `// @note <text>` (e.g. `// @note chosen for performance`, `// @note aligned with existing pattern`). If the symbol already has a JSDoc block, put the note inside that block instead of a separate `// @note`.
> - **Minimal comments.** Code self-documents — comment only the non-obvious (tricky invariant, security reason, deliberate deviation). Do not narrate obvious code.
> - **Do not write or scaffold tests** unless the user explicitly asks for them.
> - **Never edit the Prisma schema** or run schema/DB-mutating commands (`db:migrate`, `db:push`, `migration:*`, `db:generate`). If a schema change is needed, stop and tell the user.
> - **Never commit, stage, or unstage** on your own — leave the git tree exactly as the user arranged it.
> - If an example is needed to clarify a suggestion, **provide the code example immediately** — do not defer or ask for confirmation first.

## Project Overview

ACK NestJS Boilerplate is an enterprise-grade authentication and authorization service built with:
- **NestJS v11** + **TypeScript** with strict mode (`strictNullChecks: true`, `noImplicitAny: true`) and path aliases
- **Prisma ORM** → MongoDB (replica set required for transactions). Never edit the schema or run `db:generate`/`db:migrate`/`db:push` yourself; if a schema change is needed, stop and tell the user
- **Redis** → cache + session store (`db:0`) and BullMQ queues (`db:1`)
- **PNPM** as the only allowed package manager (npm/yarn blocked; enforced by preinstall script)
- **Node.js** >= 24.11.0
- **ES256** (access token) / **ES512** (refresh token) JWT algorithms
- **Repository Design Pattern** for data access layer
- **Modular Architecture** with clear separation of concerns
- **SOLID Principles** throughout the codebase
- **class-validator** and **class-transformer** for DTO validation and transformation
- **Swagger** for API documentation (disabled when `APP_ENV=production`)
- **i18n** for internationalization with nested JSON structure
- Sessions use dual storage (Redis + Database) for performance and management

## Architecture

### Module Structure

No module contains every folder. Each module includes only the folders its feature needs, drawn from these tiers:
```
module/
# Core (present in almost every module)
├── constants/          # Static values and configuration
├── controllers/        # API endpoint handlers
├── dtos/               # Data Transfer Objects with validation
├── enums/              # Type-safe enumerations
├── interfaces/         # TypeScript contracts
├── repositories/       # Data access layer (Prisma)
├── services/           # Business logic
├── utils/              # Helper utilities
# Common (present when the feature needs them)
├── decorators/         # Custom metadata decorators
├── docs/               # Swagger/OpenAPI documentation decorators
├── guards/             # Authorization and access control
# Specialized (a few modules only)
├── factories/          # Object creation patterns (policy)
├── indicators/         # Health-check indicators (health)
├── interceptors/       # Request/response transformation (activity-log)
├── processors/         # Background job handlers / BullMQ (notification)
├── templates/          # Email/document templates (notification, term-policy)
└── validations/        # Custom validators (auth, feature-flag)
```

### Repository Design Pattern

- `Repository` → data access only, injects `DatabaseService` directly (no `@Inject`)
- `Service` → business logic only, injects repository as class (no interface for repo)
- `Service` always implements an **interface** (`IUserService`)
- Never inject `DatabaseService` directly into services

```typescript
// ✅ Repository — no interface needed
@Injectable()
export class UserRepository {
    constructor(private readonly databaseService: DatabaseService) {}

    async findById(id: string): Promise<User | null> {
        return this.databaseService.user.findFirst({
            where: { id }
        });
    }
}

// ✅ Service Interface — services always have an interface
export interface IUserService {
    findById(id: string): Promise<User | null>;
}

// ✅ Service Implementation — injects repo as class
export class UserService implements IUserService {
    constructor(private readonly userRepository: UserRepository) {}
}
```

### Path Aliases (always use, never relative paths)

- `@app/*` → `src/app/*`
- `@common/*` → `src/common/*`
- `@config` → `src/configs/index.ts`
- `@configs/*` → `src/configs/*`
- `@modules/*` → `src/modules/*`
- `@routes/*` → `src/router/routes/*`
- `@router` → `src/router/router.module.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrechristikan/ack-nestjs-boilerplate](https://github.com/andrechristikan/ack-nestjs-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
