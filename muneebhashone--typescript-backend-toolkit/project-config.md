---
trigger: always_on
description: Zod schema patterns for validation and OpenAPI documentation
---


# Zod Schema Patterns

## Core Principle

Every module should have a schema file that defines request/response validation using Zod schemas.

## Import Pattern

```typescript
import { z } from 'zod';
import validator from 'validator';
```

## Complete Response Schema Workflow

**IMPORTANT:** This section shows the complete pattern for defining response schemas, using them in routers, and typing controllers.

### Step 1: Define in Schema File (`*.schema.ts`)

```typescript
import { z } from 'zod';
import validator from 'validator';
import { R } from '@/plugins/magic/response.builders';
import { itemOutSchema } from './item.dto';

// Request validation schemas
export const createItemSchema = z.object({
  name: z.string({ required_error: 'Name is required' }).min(1).max(100),
  description: z.string().min(10).max(500).optional(),
  status: z.enum(['active', 'inactive']).default('active'),
});

export const updateItemSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  description: z.string().min(10).max(500).optional(),
  status: z.enum(['active', 'inactive']).optional(),
});

// Response schemas using R builders
export const createItemResponseSchema = R.success(itemOutSchema);
export const getItemByIdResponseSchema = R.success(itemOutSchema);
export const getItemsResponseSchema = R.paginated(itemOutSchema);
export const updateItemResponseSchema = R.success(itemOutSchema);

// Export request types
export type CreateItemSchemaType = z.infer<typeof createItemSchema>;
export type UpdateItemSchemaType = z.infer<typeof updateItemSchema>;

// Export response types (PascalCase for types)
export type CreateItemResponseSchema = z.infer<typeof createItemResponseSchema>;
export type GetItemByIdResponseSchema = z.infer<typeof getItemByIdResponseSchema>;
export type GetItemsResponseSchema = z.infer<typeof getItemsResponseSchema>;
export type UpdateItemResponseSchema = z.infer<typeof updateItemResponseSchema>;
```

### Step 2: Use in Router (`*.router.ts`)

```typescript
import { MagicRouter } from '@/plugins/magic/router';
import { canAccess } from '@/middlewares/can-access';
import { 
  createItemSchema,
  updateItemSchema,
  createItemResponseSchema,
  getItemsResponseSchema,
} from './item.schema';
import { handleCreate, handleGetItems } from './item.controller';

const router = new MagicRouter();

router.post('/', {
  requestType: { body: createItemSchema },
  responses: { 201: createItemResponseSchema }, // Use response schema
}, canAccess(), handleCreate);

router.get('/', {
  requestType: { query: listQuerySchema },
  responses: { 200: getItemsResponseSchema }, // Use response schema
}, canAccess(), handleGetItems);
```

### Step 3: Type Controller (`*.controller.ts`)

```typescript
import type { Request } from 'express';
import type { ResponseExtended } from '@/types';
import type {
  CreateItemSchemaType,
  CreateItemResponseSchema,
} from './item.schema';
import { createItem } from './item.service';

export const handleCreate = async (
  req: Request<unknown, unknown, CreateItemSchemaType>,
  res: ResponseExtended<CreateItemResponseSchema>, // Typed response
) => {
  const item = await createItem(req.body);
  
  return res.created?.({ // Type-safe response
    success: true,
    message: 'Item created',
    data: item,
  });
};
```

**Benefits of this workflow:**
- ✅ End-to-end type safety from request to response
- ✅ Accurate OpenAPI documentation generation
- ✅ Runtime validation (optional)
- ✅ IDE autocomplete for response structure
- ✅ Consistent response formats across API

## Schema Structure

Schemas are exported directly, NOT wrapped in request/response objects:

```typescript
import { z } from 'zod';
import validator from 'validator';

export const createItemSchema = z.object({
  name: z.string({ required_error: 'Name is required' }).min(1).max(100),
  description: z
    .string({ required_error: 'Description is required' })
    .min(10)
    .max(500),
  status: z.enum(['active', 'inactive']).default('active'),
  categoryId: z
    .string({ required_error: 'Category ID is required' })
    .refine((value) => validator.isMongoId(value), 'Category ID must be valid'),
});

export const updateItemSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  description: z.string().min(10).max(500).optional(),
  status: z.enum(['active', 'inactive']).optional(),
});
```

## Common Patterns

### String Validation with Required Error

```typescript
z.string({ required_error: 'Field name is required' }).min(1).max(64);
```

### Email Validation

```typescript
z.string({ required_error: 'Email is required' }).email({
  message: 'Email is not valid',
});
```

### MongoDB ObjectId Validation

Use validator package, NOT regex:

```typescript
z.string({ required_error: 'ID is required' })
  .min(1)
  .refine((value) => validator.isMongoId(value), 'ID must be valid');
```

### Alphanumeric Validation

```typescript
z.string({ required_error: 'Code is required' })
  .min(4)
  .max(4)
  .refine((value) => validator.isAlphanumeric(value), 'Code must be valid');
```

### Query Parameters with Transform

```typescript
export const listItemsQuerySchema = z.object({
  searchString: z.string().optional(),
  limitParam: z
    .string()
    .default('10')
    .refine(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
