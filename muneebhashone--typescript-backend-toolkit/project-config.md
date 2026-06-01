---
trigger: always_on
description: Step-by-step guide for creating a new module using the tbk CLI
---


# Creating a New Module

This guide shows how to create a new module using the `tbk` CLI tool and customize it according to project patterns.

## Quick Start

### Step 1: Generate Module Scaffolding

Use the `tbk` CLI to generate all module files automatically:

```bash
pnpm exec tbk generate:module <module-name>
```

Or with custom API path prefix:

```bash
pnpm exec tbk generate:module <module-name> --path /api/v1
```

**Example:**

```bash
pnpm exec tbk generate:module product
# Creates: src/modules/product/ with all required files
```

This creates a complete module structure:

```
src/modules/<module-name>/
├── <module-name>.dto.ts         # TypeScript types and Zod schemas
├── <module-name>.model.ts       # Mongoose model
├── <module-name>.schema.ts      # Request/response validation schemas
├── <module-name>.services.ts    # Business logic and database operations
├── <module-name>.controller.ts  # HTTP request handlers
└── <module-name>.router.ts      # MagicRouter route definitions
```

### Step 2: Customize Module Files

The generated files follow project patterns but need customization for your specific use case. Refer to these rules for detailed patterns:

#### 2.1 Update Model (`<module-name>.model.ts`)

- **Rule:** `@models`
- Add/modify fields in the Mongoose schema
- Define indexes, virtuals, and methods
- Configure schema options (timestamps, etc.)

#### 2.2 Update DTOs (`<module-name>.dto.ts`)

- Define input/output types using Zod
- Use `definePaginatedResponse` from `common.utils` for list endpoints
- Export type definitions for type safety

#### 2.3 Update Validation Schemas (`<module-name>.schema.ts`)

- **Rule:** `@schemas`
- Add/modify Zod validation for create/update operations
- Configure query parameter validation (pagination, search, filters)
- Define proper error messages and transformations

#### 2.4 Update Services (`<module-name>.services.ts`)

- **Rule:** `@services`
- Implement business logic
- Handle database operations using the model
- Use proper error handling (throw errors with descriptive messages)
- Optimize queries with proper filtering, pagination, and sorting

#### 2.5 Update Controller (`<module-name>.controller.ts`)

- **Rule:** `@controllers`
- Handle HTTP request/response
- Use `successResponse` from `@/utils/response.utils`
- Use proper HTTP status codes from `@/openapi/status-codes`
- Keep controllers thin - delegate logic to services

#### 2.6 Update Router (`<module-name>.router.ts`)

- **Rule:** `@routing`
- Configure MagicRouter routes
- Add proper middleware (authentication, authorization)
- Use `canAccess()` for protected routes
- Define request validation schemas

### Step 3: Register Router

Add the router to `src/routes/routes.ts`:

```typescript
import <module-name>Router from '@/modules/<module-name>/<module-name>.router';

// In the registerRoutes function or where routes are registered
app.use(<module-name>Router);
```

### Step 4: Test the Module

1. Start development server:

   ```bash
   pnpm dev
   ```

2. Visit Swagger UI:

   ```
   http://localhost:3000/docs
   ```

3. Test all endpoints using the interactive API documentation

4. Verify:
   - All CRUD operations work correctly
   - Validation catches invalid inputs
   - Error responses are properly formatted
   - OpenAPI documentation is accurate

## Module File Responsibilities

### 1. DTO (`*.dto.ts`)

- Zod schemas for input/output validation
- TypeScript type definitions
- Paginated response schemas

### 2. Model (`*.model.ts`)

- Mongoose schema definition
- Database field types and constraints
- Indexes and virtuals
- Model interface extending Document

### 3. Schema (`*.schema.ts`)

- Request validation schemas (create, update, query)
- Zod transformations and refinements
- Type exports for controllers

### 4. Services (`*.services.ts`)

- Business logic implementation
- Database operations (CRUD)
- Data transformation
- Error handling

### 5. Controller (`*.controller.ts`)

- HTTP request/response handling
- Call service methods
- Return standardized responses
- Handle HTTP status codes

### 6. Router (`*.router.ts`)

- Route definitions using MagicRouter
- Middleware configuration
- Request validation binding
- OpenAPI metadata

## Best Practices

### Follow Project Patterns

- **Always** use MagicRouter for automatic OpenAPI generation
- **Never** use plain Express `app.get()` or `router.get()`
- **Always** validate requests with Zod schemas
- **Always** use TypeScript strict mode - no `any` types

### Error Handling

- Throw descriptive errors in services
- Let global error handler format responses
- Use proper HTTP status codes

### Type Safety

- Export and use TypeScript types from DTOs
- Use Zod's `.infer` for type generation
- Keep runtime validation and TypeScript types in sync

### Code Organization

- Keep controllers thin - delegate to services
- Put business logic in services
- Use common utilities for shared functionality
- Follow the single responsibility principle

## Advanced Customization

### Adding Authentication

Use `canAccess()` middleware in router:

```typescript
import { canAccess } from '@/middlewares/can-access';

router.post(
  '/',
  { requestType: { body: createSchema } },
  canAccess(), // Add authentication
  handleCreate,
);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
