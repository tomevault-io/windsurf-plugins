---
trigger: always_on
description: Routing patterns using MagicRouter for automatic OpenAPI generation
---


# Routing with MagicRouter

## Core Principle

NEVER use plain Express routing. ALWAYS use MagicRouter from [router.ts](mdc:src/plugins/magic/router.ts).

## Pattern Template

```typescript
import MagicRouter from '@/plugins/magic/router';
import { canAccess } from '@/middlewares/can-access';
import {
  handleAction,
  handleGetById,
  handleCreate,
  handleSearch,
} from './module.controller';
import {
  actionSchema,
  createSchema,
  idParamsSchema,
  searchQuerySchema,
  actionResponseSchema,
  getMeResponseSchema,
  createItemResponseSchema,
  getItemByIdResponseSchema,
  searchItemsResponseSchema,
} from './module.schema';

export const MODULE_ROUTER_ROOT = '/module';

const moduleRouter = new MagicRouter(MODULE_ROUTER_ROOT);

// Public route with schema validation and response config
moduleRouter.post(
  '/action',
  {
    requestType: { body: actionSchema },
    responses: {
      200: actionResponseSchema,
    },
  },
  handleAction,
);

// Protected route with authentication
moduleRouter.get(
  '/me',
  {
    responses: {
      200: getMeResponseSchema,
    },
  },
  canAccess(),
  handleGetById,
);

// Protected route with schema, auth, and response config
moduleRouter.post(
  '/create',
  {
    requestType: { body: createSchema },
    responses: {
      201: createItemResponseSchema,
    },
  },
  canAccess(),
  handleCreate,
);

// Route with params
moduleRouter.get(
  '/:id',
  {
    requestType: { params: idParamsSchema },
    responses: {
      200: getItemByIdResponseSchema,
    },
  },
  handleGetById,
);

// Route with query params (paginated)
moduleRouter.get(
  '/search',
  {
    requestType: { query: searchQuerySchema },
    responses: {
      200: searchItemsResponseSchema,
    },
  },
  handleSearch,
);

export default moduleRouter.getRouter();
```

## MagicRouter API

### Router Instantiation

```typescript
const router = new MagicRouter(ROUTER_ROOT);
```

- Create router instance with root path (e.g., `/auth`, `/user`)
- Root path used for route grouping and OpenAPI tag generation

### Route Definition Signature

```typescript
router.method(path, requestType, ...handlers);
```

**Parameters:**

1. `path`: Route path string (e.g., `/login`, `/:id`)
2. `requestType`: Schema configuration object
3. `...handlers`: Middleware functions and controller (spread arguments)

### Request Type Object

```typescript
{
  requestType?: {
    body?: ZodSchema,      // Request body validation
    params?: ZodSchema,    // URL params validation
    query?: ZodSchema,     // Query string validation
  },
  responses?: {            // NEW: Response schemas per status code
    200?: ResponseSchema,  // Success response
    201?: ResponseSchema,  // Created response
    404?: ResponseSchema,  // Not found response
    // ... other status codes
  },
  contentType?: string,    // 'application/json' | 'multipart/form-data' | etc.
}
```

- Use empty object `{}` when no validation needed
- Can combine `body`, `params`, and `query` in same route
- **NEW**: Add `responses` object for response schemas (RECOMMENDED)

### Response Configuration (NEW - RECOMMENDED)

**BEST PRACTICE:** Define response schemas in your schema file and import them:

```typescript
// In module.schema.ts
import { R } from '@/plugins/magic/response.builders';
import { itemOutSchema } from './module.dto';

export const createItemResponseSchema = R.success(itemOutSchema);
export const getItemsResponseSchema = R.paginated(itemOutSchema);

export type CreateItemResponseSchema = z.infer<typeof createItemResponseSchema>;
export type GetItemsResponseSchema = z.infer<typeof getItemsResponseSchema>;

// In module.router.ts
import { createItemResponseSchema, getItemsResponseSchema } from './module.schema';

responses: { 201: createItemResponseSchema }
responses: { 200: getItemsResponseSchema }
```

**Alternative (inline):** Use response builders directly in router:

```typescript
import { R } from '@/plugins/magic/response.builders';

// Standard success response
responses: { 200: R.success(itemSchema) }

// Paginated list response
responses: { 200: R.paginated(itemSchema) }

// Created response
responses: { 201: R.success(itemSchema) }

// No content response
responses: { 204: R.noContent() }

// Error response
responses: { 404: R.error() }

// Multiple status codes
responses: {
  200: R.success(itemSchema),
  404: R.error(),
}

// Raw response (non-envelope)
responses: { 200: R.raw(customSchema) }
```

**Response Builders:**

- `R.success(schema)` - Standard envelope: `{ success, message?, data? }`
- `R.paginated(itemSchema)` - Paginated list: `{ success, message?, data: { items, paginator } }`
- `R.noContent()` - Empty 204 response
- `R.error(schema?)` - Error envelope (optional custom schema)
- `R.raw(schema)` - Non-envelope response (e.g., healthcheck)

**Why define in schema files?**

- ✅ Type-safe controller responses with `ResponseExtended<T>`
- ✅ Centralized response definitions
- ✅ Easier to maintain and update
- ✅ Better code organization

### Handler Order

The last handler in the spread is treated as the **controller**. All preceding handlers are **middleware**.

```typescript
// Public route
router.post('/action', { requestType: { body: schema } }, controller);

// With one middleware

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
