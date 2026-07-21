---
trigger: always_on
description: This file provides AI coding agents with essential context for working on the nestjs-dynamoose project.
---

# AGENTS.md - NestJS-Dynamoose Development Guide

This file provides AI coding agents with essential context for working on the nestjs-dynamoose project.

## Project Overview

**nestjs-dynamoose** is a NestJS module that integrates [Dynamoose](https://dynamoosejs.com/) (a DynamoDB ODM) with NestJS's dependency injection system. It provides decorators, providers, and utilities to seamlessly use DynamoDB models in NestJS applications.

**Key Purpose:** Bridge the gap between Dynamoose's ODM functionality and NestJS's module/DI architecture.

## Architecture & Design Patterns

### Module Structure

The project follows NestJS's standard dynamic module pattern with global and feature modules:

- **`DynamooseCoreModule`** (Global): Handles Dynamoose initialization, AWS configuration, and global setup
- **`DynamooseModule`** (Feature): Provides model registration via static methods:
  - `forRoot(options)` / `forRootAsync(options)` - Global configuration
  - `forFeature(models)` / `forFeatureAsync(factories)` - Model registration per feature module

### Dependency Injection Pattern

Models are injected using a token-based system:

```typescript
@InjectModel('User') 
private userModel: Model<User, UserKey>
```

The `getModelToken(name)` utility in `lib/common/dynamoose.utils.ts` creates consistent tokens for model providers.

### Key Components

| File | Purpose |
|------|---------|
| `lib/dynamoose.module.ts` | Main module with static registration methods |
| `lib/dynamoose-core.module.ts` | Global module handling Dynamoose initialization |
| `lib/dynamoose.providers.ts` | Creates model providers with proper DI |
| `lib/common/dynamoose.decorators.ts` | Exports `@InjectModel()` decorator |
| `lib/common/dynamoose.utils.ts` | Token generation utilities |
| `lib/interfaces/` | TypeScript interfaces for module options, models, transactions |

## Development Workflows

### Build & Quality Scripts

```bash
npm run build    # Compile TypeScript (lib/ → dist/)
npm run lint     # ESLint with Prettier integration
npm run format   # Prettier formatting for lib/**/*.ts
```

### Build Configuration

- **Source:** `lib/` directory (NOT `src/`)
- **Output:** `dist/` directory
- **Entry point:** `index.ts` exports `./dist`
- **TypeScript:** CommonJS modules, decorators enabled, strict mode
- **Target:** ES2021, Node.js compatible

### CI/CD Pipeline

- **GitHub Actions:** Node.js 22.17.1
- **Steps:** `npm ci` → `npm run lint` → `npm run build`
- **Pre-commit hooks:** 
  - `lint-staged` runs Prettier on `*.ts` files
  - `commitlint` enforces Angular commit conventions

### Commit Message Format

Follow Angular conventions:

```
<type>(<scope>): <subject>

Examples:
- feature(core): add support for Dynamoose v4
- bugfix(providers): fix async model factory injection
- docs(readme): improve transaction example
```

Types: `build`, `ci`, `docs`, `feature`, `bugfix`, `perf`, `refactor`, `style`, `test`

## Project-Specific Conventions

### File Organization

```
lib/
├── common/               # Decorators & utilities
├── interfaces/           # TypeScript interfaces with barrel exports
├── dynamoose.module.ts   # Main module
├── dynamoose-core.module.ts
├── dynamoose.providers.ts
├── dynamoose.constants.ts
└── index.ts              # Public API
```

### TypeScript Patterns

**Interface Segregation:** Separate key interfaces from document interfaces

```typescript
// Key interface (hash/range keys only)
export interface UserKey {
  id: string;
}

// Document interface (all attributes)
export interface User extends UserKey {
  name: string;
  email?: string;
}
```

**Type Safety with Dynamoose:** 
- Extensive use of `any` types for Dynamoose compatibility
- ESLint rules disabled: `@typescript-eslint/no-unsafe-*`, `@typescript-eslint/no-explicit-any`
- This is intentional due to Dynamoose's dynamic nature

### Module Registration Patterns

**Synchronous:**
```typescript
DynamooseModule.forFeature([{
  name: 'User',
  schema: UserSchema,
  options: { tableName: 'user' }
}])
```

**Async with DI:**
```typescript
DynamooseModule.forFeatureAsync([{
  name: 'User',
  useFactory: (_, configService: ConfigService) => ({
    schema: UserSchema,
    options: { tableName: configService.get('USER_TABLE_NAME') }
  }),
  inject: [ConfigService]
}])
```

⚠️ **Important:** First parameter of `useFactory` is reserved (ignore with `_,`)

### Transaction Support

The `TransactionSupport` abstract class provides transaction capabilities:

```typescript
@Injectable()
export class UserService extends TransactionSupport {
  constructor(
    @InjectModel('User') private userModel: Model<User, UserKey>,
    @InjectModel('Account') private accountModel: Model<Account, AccountKey>
  ) {
    super();
  }

  async createUserWithAccount(user: User, account: Account) {
    await this.transaction([
      this.userModel.transaction.create(user),
      this.accountModel.transaction.create(account)
    ]);
  }
}
```

## Dependencies & Peer Dependencies

### Core Dependencies

- **dynamoose:** ^3.2.0 || ^4.0.0 - DynamoDB ODM
- **@nestjs/common & @nestjs/core:** ^8.0.0 || ^9.0.0 || ^10.0.0 || ^11.0.0
- **@aws-sdk/client-dynamodb:** ^3.0.0
- **rxjs:** ^6.0.0 || ^7.0.0
- **reflect-metadata:** ^0.1.13 || ^0.2.0


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hardyscc/nestjs-dynamoose](https://github.com/hardyscc/nestjs-dynamoose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
