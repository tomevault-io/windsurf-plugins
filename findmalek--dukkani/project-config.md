---
trigger: always_on
description: Entities transform database data to output schemas. Each entity has three files:
---

# Code Patterns

## Entity Pattern

Entities transform database data to output schemas. Each entity has three files:

### Structure

```
packages/common/src/entities/{entity-name}/
  ├── entity.ts      # Transformation logic (getSimpleRo, getRo)
  ├── query.ts       # Prisma query builders (getInclude, getWhere, getOrder)
  └── index.ts       # Exports
```

### Entity Class Pattern

```typescript
// entity.ts
export class StoreEntity {
  static getSimpleRo(entity: StoreSimpleDbData): StoreSimpleOutput {
    return { id: entity.id, name: entity.name };
  }

  static getRo(entity: StoreIncludeDbData): StoreIncludeOutput {
    return {
      ...this.getSimpleRo(entity),
      owner: UserEntity.getSimpleRo(entity.owner),
      products: entity.products.map(ProductEntity.getSimpleRo),
    };
  }
}
```

### Query Class Pattern

```typescript
// query.ts
export class StoreQuery {
  static getSimpleInclude() {
    return {} satisfies Prisma.StoreInclude;
  }

  static getInclude() {
    return {
      ...this.getSimpleInclude(),
      owner: UserQuery.getSimpleInclude(),
      products: ProductQuery.getSimpleInclude(),
    } satisfies Prisma.StoreInclude;
  }

  static getWhere(storeIds: string[], filters?: FilterOptions) {
    return {
      id: { in: storeIds },
      ...(filters?.search && { name: { contains: filters.search } }),
    } satisfies Prisma.StoreWhereInput;
  }

  static getOrder(direction: "asc" | "desc", field: string) {
    return { [field]: direction } satisfies Prisma.StoreOrderByWithRelationInput;
  }
}
```

## Schema Pattern

Schemas define input/output validation using Zod.

### Structure

```
packages/common/src/schemas/{entity-name}/
  ├── input.ts       # Input validation schemas
  ├── output.ts      # Output type schemas
  ├── enums.ts       # Enum schemas
  └── index.ts       # Exports
```

### Input Schema Pattern

```typescript
export const createStoreInputSchema = z.object({
  name: z.string().min(1),
  slug: z.string().min(1),
  description: z.string().optional(),
});

export const listStoresInputSchema = z.object({
  page: z.number().int().min(1).default(1),
  limit: z.number().int().min(1).max(100).default(10),
  search: z.string().optional(),
});

export type CreateStoreInput = z.infer<typeof createStoreInputSchema>;
export type ListStoresInput  = z.infer<typeof listStoresInputSchema>;
```

## Service Pattern

Services contain business logic and database operations.

```typescript
// packages/common/src/services/{entity}-service.ts
export class StoreService {
  static async getAllStores(userId: string): Promise<StoreSimpleOutput[]> {
    const stores = await database.store.findMany({
      where: { ownerId: userId },
      include: StoreQuery.getClientSafeInclude(),
    });
    return stores.map(StoreEntity.getSimpleRo);
  }
}
```

## Router Pattern

Routers define API endpoints using oRPC.

```typescript
// packages/orpc/src/routers/{entity}.ts
export const storeRouter = {
  getAll: protectedProcedure
    .input(listStoresInputSchema.optional())
    .handler(async ({ context }) => {
      return await StoreService.getAllStores(context.session.user.id);
    }),

  getById: protectedProcedure
    .input(getStoreInputSchema)
    .handler(async ({ input, context }) => {
      return await StoreService.getStoreById(input.id, context.session.user.id);
    }),
};
```

## Query and Mutation Pattern

Queries and mutations are accessed through centralized `appQueries` and `appMutations` in `shared/api/`.
Never call `api.x.queryOptions` directly in components — always go through `appQueries`.

### Query pattern

```typescript
import { useQuery, useSuspenseQuery } from "@tanstack/react-query";
import { appQueries } from "@/shared/api/queries";

const { data, isLoading } = useQuery(appQueries.order.all({ input: { page, limit } }));
const { data: product }   = useSuspenseQuery(appQueries.product.byId({ input: { id } }));
```

### Mutation pattern

```typescript
import { useMutation } from "@tanstack/react-query";
import { appMutations } from "@/shared/api/mutations";

// appMutations handle cache invalidation automatically.
// Any onSuccess/onError you pass runs AFTER the built-in behavior — it's chained, not replaced.
const createMutation = useMutation(appMutations.product.create());

const deleteMutation = useMutation(
  appMutations.product.delete({
    onSuccess: () => router.push("/products"),  // ← safe: runs after invalidation
  })
);

// ❌ Never spread and override onSuccess — it drops the built-in invalidation
const bad = useMutation({
  ...appMutations.product.create(),
  onSuccess: () => {},  // ← this silently removes cache invalidation
});
```

## Query Data Type Extraction

Use `QueryData<T>` to extract the resolved data type from any `appQueries` factory at the type level:

```typescript
import type { QueryData } from "@/shared/api/queries";

// Extract types without calling the function or importing server-only code
type ProductList = QueryData<typeof appQueries.product.all>;
type CurrentUser = QueryData<typeof appQueries.account.currentUser>;
type StoreList   = QueryData<typeof appQueries.store.all>;

// Use in component props
interface Props {
  product: QueryData<typeof appQueries.product.byId>;
}

// Use to annotate return values

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FindMalek/dukkani](https://github.com/FindMalek/dukkani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
