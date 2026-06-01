---
trigger: always_on
description: Controller patterns for handling business logic
---


# Controller Patterns

## Core Principle

Controllers are async functions that handle validated requests and return responses. They should be thin - delegate complex logic to services.

## Controller Template

```typescript
import type { Request } from 'express';
import type { ResponseExtended } from '@/types';
import type {
  CreateItemSchemaType,
  GetItemsSchemaType,
  UpdateItemSchemaType,
  CreateItemResponseSchema,
  GetItemByIdResponseSchema,
  GetItemsResponseSchema,
  UpdateItemResponseSchema,
} from './module.schema';
import {
  createItem,
  deleteItem,
  findById,
  getItems,
  updateItem,
} from './module.service';

/**
 * Create new item
 */
export const handleCreate = async (
  req: Request<unknown, unknown, CreateItemSchemaType>,
  res: ResponseExtended<CreateItemResponseSchema>,
) => {
  // 1. Extract validated data (already validated by Zod middleware)
  const data = req.body;

  // 2. Access JWT payload (if route uses canAccess middleware)
  const userId = req.user?.sub;

  // 3. Call service layer for business logic
  const item = await createItem({ ...data, createdBy: userId });

  // 4. Return typed response
  return res.created?.({
    success: true,
    message: 'Item created successfully',
    data: item,
  });
};

/**
 * Get single item by ID
 */
export const handleGetById = async (
  req: Request<{ id: string }, unknown, unknown>,
  res: ResponseExtended<GetItemByIdResponseSchema>,
) => {
  const { id } = req.params;

  const item = await findById(id);

  if (!item) {
    return res.notFound?.({
      success: false,
      message: 'Item not found',
    });
  }

  return res.ok?.({
    success: true,
    data: item,
  });
};

/**
 * Get paginated list of items
 */
export const handleGetItems = async (
  req: Request<unknown, unknown, unknown, GetItemsSchemaType>,
  res: ResponseExtended<GetItemsResponseSchema>,
) => {
  const { results, paginatorInfo } = await getItems(req.query);

  return res.ok?.({
    success: true,
    data: {
      items: results,
      paginator: paginatorInfo,
    },
  });
};

/**
 * Update existing item
 */
export const handleUpdate = async (
  req: Request<{ id: string }, unknown, UpdateItemSchemaType>,
  res: ResponseExtended<UpdateItemResponseSchema>,
) => {
  const { id } = req.params;
  const data = req.body;
  const userId = req.user?.sub;

  const item = await updateItem(id, data, userId);

  if (!item) {
    return res.notFound?.({
      success: false,
      message: 'Item not found',
    });
  }

  return res.ok?.({
    success: true,
    message: 'Item updated successfully',
    data: item,
  });
};

/**
 * Delete item
 */
export const handleDelete = async (
  req: Request<{ id: string }, unknown, unknown>,
  res: ResponseExtended<Record<string, never>>,
) => {
  const { id } = req.params;

  await deleteItem(id);

  return res.ok?.({
    success: true,
    message: 'Item deleted successfully',
  });
};
```

## Key Points

### TypeScript Request Typing

Always use TypeScript generics for type-safe requests:

```typescript
Request<ParamsType, unknown, BodyType, QueryType>;

// Examples:
Request<{ id: string }, unknown, unknown>; // params only
Request<unknown, unknown, CreateUserSchemaType>; // body only
Request<unknown, unknown, unknown, GetUsersSchemaType>; // query only
Request<{ id: string }, unknown, UpdateUserSchemaType>; // params + body
```

### Request Data Access

- `req.body` - Request body (validated by Zod)
- `req.params` - URL parameters (validated by Zod)
- `req.query` - Query parameters (validated by Zod)
- `req.user` - JWT token payload (if using extractJwt middleware)
- `req.file` / `req.files` - Uploaded files (if using multer middleware)

### JWT Payload Access

When route uses `extractJwt` middleware from [extract-jwt-schema.ts](mdc:src/middlewares/extract-jwt-schema.ts):

```typescript
import type { JwtPayload } from '@/utils/jwt.utils';

// Access JWT payload via req.user
const userId = req.user?.sub; // User ID
const email = req.user?.email; // User email
const username = req.user?.username; // Username
const role = req.user?.role; // User role

// Type assertion if needed
const payload = req.user as JwtPayload;
```

**JwtPayload Type:**

```typescript
type JwtPayload = {
  sub: string; // User ID
  email?: string | null;
  phoneNo?: string | null;
  username: string;
  role: RoleType;
};
```

### File Upload Access

When route uses multer middleware from [multer-s3.ts](mdc:src/middlewares/multer-s3.ts):

```typescript
const file = req.file; // For single file
const files = req.files; // For multiple files
const url = (req.file as any).location; // S3 URL
```

### Response Pattern (RECOMMENDED)

Use `ResponseExtended` typed response helpers for all controllers:

```typescript
import type { ResponseExtended } from '@/types';
import type { CreateItemResponseSchema } from './module.schema';

export const handleCreate = async (
  req: Request<unknown, unknown, CreateItemSchemaType>,
  res: ResponseExtended<CreateItemResponseSchema>,
) => {
  const item = await createItem(req.body);
  
  // 201 Created response
  return res.created?.({
    success: true,
    message: 'Item created',
    data: item,
  });
};
```

**Available Response Methods:**

```typescript
// 200 OK - Success response
return res.ok?.({
  success: true,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
