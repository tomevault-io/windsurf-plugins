---
trigger: always_on
description: Rules for the NestJS API application (apps/api)
---


# Cursor Agent Rules for CaseAI Connect - API

## NestJS Dependency Injection

### Import Type Rule
When NestJS requires runtime access to classes for dependency injection (services, controllers, guards, etc.), you MUST use regular imports, not type-only imports.

**Rule**: If you get a NestJS DI error about a class being undefined at runtime, it means you used `import type` instead of `import`.

**Solution**: 
- Change `import type { MyService } from './my.service'` to `import { MyService } from './my.service'`
- Add a biome-ignore comment: `// biome-ignore lint/style/useImportType: Required at runtime for NestJS DI`

**When to use**:
- Services injected via `@InjectRepository()`, `@Inject()`, or constructor injection
- Controllers, Guards, Interceptors, Pipes
- Any class that needs to be available at runtime for NestJS DI

**When type-only imports are OK**:
- DTOs, interfaces, types
- Return types, parameter types
- Anything not used for DI

**Example**:
```typescript
// ❌ Wrong - will cause DI error
import type { UsersService } from "@/users/users.service"

// ✅ Correct
// biome-ignore lint/style/useImportType: Required at runtime for NestJS DI
import { UsersService } from "@/users/users.service"
```

## DTO Organization

### Domain-Based DTO Location

**Rule**: DTOs (Data Transfer Objects) MUST be co-located with their domain logic in `packages/api-contracts/src/{domain}/` as a single consolidated file per domain.

**Structure**:
- All DTOs for a domain MUST be consolidated into a single file: `packages/api-contracts/src/{domain}/{domain}.dto.ts` (e.g., `packages/api-contracts/src/projects/projects.dto.ts`, `packages/api-contracts/src/me/me.dto.ts`)
- **DO NOT** create separate files for each DTO (e.g., `create-project.dto.ts`, `list-projects.dto.ts`, etc.)
- All DTOs are exported from `packages/api-contracts/src/index.ts` for consumption by both API and frontend
- Controllers and routes import DTOs from `@caseai-connect/api-contracts`

**Why this pattern?**
- **Reduced file proliferation**: Avoids having many small DTO files that are hard to navigate
- **Better organization**: All related DTOs for a domain are in one place
- **Easier maintenance**: Related types are easier to find and update together
- **Cohesion**: DTOs are tightly coupled to domain logic and API contracts
- **Single source of truth**: DTOs live in a shared package accessible to all consumers
- **No circular dependencies**: API-contracts is a separate package that API and frontend both depend on
- **Frontend integration**: Frontend imports directly from `@caseai-connect/api-contracts`

**Import Rules**:
- **Within API**: Controllers and routes import DTOs from `@caseai-connect/api-contracts`
- **For frontend**: Frontend imports from `@caseai-connect/api-contracts`

**Example - Creating DTOs**:

1. **Create consolidated DTO file** (`packages/api-contracts/src/projects/projects.dto.ts`):
```typescript
// Project entity DTO
export type ProjectDto = {
  id: string
  name: string
  organizationId: string
  createdAt: number
  updatedAt: number
}

// Create project DTOs
export type CreateProjectRequestDto = {
  name: string
  organizationId: string
}

export type CreateProjectResponseDto = {
  id: string
  name: string
  organizationId: string
}

// List projects DTOs
export type ListProjectsResponseDto = {
  projects: ProjectDto[]
}

// Update project DTOs
export type UpdateProjectRequestDto = {
  name: string
}

export type UpdateProjectResponseDto = {
  id: string
  name: string
  organizationId: string
}
```

2. **Export from central index** (`packages/api-contracts/src/index.ts`):
```typescript
export type {
  CreateProjectRequestDto,
  CreateProjectResponseDto,
  ListProjectsResponseDto,
  ProjectDto,
  UpdateProjectRequestDto,
  UpdateProjectResponseDto,
} from "./projects/projects.dto"
```

3. **Import in routes** (`packages/api-contracts/src/projects/projects.routes.ts`):
```typescript
import type { CreateProjectRequestDto, CreateProjectResponseDto, ListProjectsResponseDto, UpdateProjectRequestDto, UpdateProjectResponseDto } from "./projects.dto"
import type { RequestPayload, ResponseData } from "../generic"
```

4. **Import in controllers** (`apps/api/src/projects/projects.controller.ts`):
```typescript
import type { CreateProjectRequestDto } from "@caseai-connect/api-contracts"
```

## Controller Guidelines

### Route Definition Strategy

All NestJS controllers MUST use the `defineRoute` strategy for type-safe route definitions. This ensures consistency and enables type inference across the codebase.

#### Step 1: Create a Routes File

Create a `*.routes.ts` file in your domain directory (e.g., `me.routes.ts`, `protected.routes.ts`).

**For GET/DELETE routes:**
```typescript
import type { ResponseData } from "../generic"
import { defineRoute } from "../helpers"
import type { MyResponseDto } from "./my.dto"

export const MyRoutes = {
  getSomething: defineRoute<ResponseData<MyResponseDto>>({
    method: "get",
    path: "my/path",  // No leading slash - it gets normalized automatically
  }),
}
```

**For POST/PUT/PATCH routes:**
```typescript
import type { RequestPayload, ResponseData } from "../generic"
import { defineRoute } from "../helpers"
import type { MyRequestDto, MyResponseDto } from "./my.dto"

export const MyRoutes = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bayesimpact/agent-studio](https://github.com/bayesimpact/agent-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
