---
trigger: always_on
description: Guidelines for Data Access Layer via Database
---

# Guidelines for Data Access Layer via Database 

## Purpose & Overview
These rules define the standard patterns for implementing CRUD operations in the data access layer. The guidelines are based on the implementation of the 'users' module and should be followed for all database operations to maintain consistency across the codebase.

## File Structure

### Shared Domain

By default, place all data access files in the shared data directory. Only use feature domains when explicitly specified in requirements.

```
src/data/[entity-name]/
├── schema.ts               # Entity schema definitions
├── create-[entity].ts      # Create operation
├── get-[entity].ts         # Get single entity
├── update-[entity].ts      # Update operation
├── delete-[entity].ts      # Delete operation
├── search-[entity]s.ts     # Search with filters
└── __test-utils__/         # Test utilities
```

### Feature Domain

When a prompt/requirement explicitly specifies that code should be organized in a feature domain, follow this structure:

```
src/features/[feature-name]/
└── _data/                  # Feature-specific data access layer
    ├── schema.ts           # Feature's schemas
    ├── create-[entity].ts  # Create operation
    ├── get-[entity].ts     # Get single entity
    ├── update-[entity].ts  # Update operation
    ├── delete-[entity].ts  # Delete operation
    ├── search-[entity]s.ts # Search with filters
    └── __test-utils__/     # Test utilities
```

## Naming Conventions

### Function Naming
- `create[Entity]Data`: For creating records
- `get[Entity]Data`: For retrieving a single record
- `update[Entity]Data`: For updating records
- `delete[Entity]Data`: For deleting records
- `search[Entity]sData`: For searching records with filters

### Type Naming
- `[Entity]`: Main entity type from schema
- `Create[Entity]`: Type for creating entity, typically omitting auto-generated fields
- `Update[Entity]`: Type for updating entity, typically partial of the main entity
- `[Operation][Entity]DataArgs`: Type for function arguments
- `[Operation][Entity]DataResponse`: Type for function return value

## Implementation Patterns

## Schema Implementation
The `schema.ts` file should define the entity's schema, types, and Zod validators:

```typescript
// schema.ts
import { type Entity } from '@/db/schema';
import { z } from '@hono/zod-openapi';

// Define schema object with Zod validators
export const [entity]SchemaObject = {
  id: z.string().uuid(),
  created_at: z.union([z.coerce.date(), z.string()]).openapi({
    example: new Date().toISOString(),
  }),
  updated_at: z.union([z.coerce.date(), z.string()]).openapi({
    example: new Date().toISOString(),
  }),
  deleted_at: z.union([z.coerce.date(), z.string()]).nullable().openapi({
    example: null,
  }),
  // ... add entity-specific fields with validation and OpenAPI examples
};

// Create Zod schema from schema object
export const [entity]Schema = z.object([entity]SchemaObject) satisfies z.ZodType<Entity>;
// Create OpenAPI schema for documentation
export const [entity]SchemaOpenApi = [entity]Schema.openapi('[Entity]');
// Create fields enum for dynamic field references
export const [entity]SchemaFields = z.enum(Object.keys([entity]SchemaObject) as [string, ...string[]]);
// Define derived types for operations
export type Create[Entity] = Omit<[Entity], 'id' | 'created_at' | 'updated_at' | 'deleted_at'>;
export type Update[Entity] = Partial<[Entity]>;
```

## Schema Registration
After creating the entity schema, it **must** be registered in the main `schema.ts` file located at `src/data/schema.ts` for OpenAPI documentation:

```typescript
// src/data/schema.ts
import { [entity]SchemaOpenApi } from './[entity-name]/schema';
// ... other schema imports

export const schemas = {
  // ... existing schemas
  [Entity]: [entity]SchemaOpenApi,
} as const;
```

This registration ensures that the schema is available for OpenAPI documentation generation and is properly included in the API documentation.

## Test Utilities
Create test utilities in the `__test-utils__` directory to help with testing:

```typescript
// __test-utils__/make-fake-entity.ts
import { type DbClient } from '@/db/create-db-client';
import { type Entity } from '@/db/schema';
import { faker } from '@faker-js/faker';

// Create a fake entity with realistic test data
export function makeFake[Entity] {
  return {
    id: faker.string.uuid(),
    created_at: faker.date.recent(),
    updated_at: faker.date.recent(),
    deleted_at: null,
    // ... entity-specific fields with realistic fake data
    ...overrides,
  } satisfies Entity;
}

// Helper to create test entities in the database
export type CreateTest[Entity]sInDBArgs = {
  dbClient: DbClient;
  values?: Partial<Entity> | Partial<Entity>[];
};

export async function createTest[Entity]sInDB({ dbClient, values }: CreateTest[Entity]sInDBArgs) {
  const fake[Entity]s = Array.isArray(values) 
    ? values.map(makeFakeEntity) 
    : makeFakeEntity(values);
    
  const created[Entity]s = await dbClient
    .insertInto('[entity_table]')
    .values(fakeEntities)
    .returningAll()
    .execute();
    
  return created[Entity]s;
}
```

### Create Operation
```typescript
// create-[entity].ts
export type Create[Entity]DataArgs = {
  dbClient: DbClient;
  values: Create[Entity];

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/constROD) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
