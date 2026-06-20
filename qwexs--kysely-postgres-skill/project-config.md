---
trigger: always_on
description: Write effective, type-safe Kysely queries for PostgreSQL. This skill should be used when working in Node.js/TypeScript backends with Kysely installed, covering query patterns, migrations, type generation, and common pitfalls to avoid.
---


# Kysely for PostgreSQL

Kysely is a type-safe TypeScript SQL query builder. This skill provides patterns for writing effective queries, managing migrations, and avoiding common pitfalls.

**Current version:** 0.28.10 (January 2025)
**Minimum TypeScript version:** 4.6+

## When to Use This Skill

Use this skill when:
- Working in a Node.js/TypeScript project with Kysely installed
- Writing database queries for PostgreSQL
- Creating or modifying database migrations
- Debugging type inference issues in Kysely queries

## What's New in 0.28.x

### Breaking Changes

1. **InferResult returns arrays**: `InferResult<T>` now returns `InsertResult[]`, `UpdateResult[]`, `DeleteResult[]`, `MergeResult[]`. For a single result, use `InferResult<T>[number]`.

2. **Mandatory .execute()**: Removed `preventAwait` — queries no longer throw an error when awaited without `.execute()`, but the result will be `undefined`. Always call `.execute()`.

3. **Removed**: `QueryResult.numUpdatedOrDeletedRows` — use `numAffectedRows` instead.

4. **TypeScript 4.5 and older are no longer supported**.

### New Features

#### Controlled Transactions with Savepoints

```typescript
// Create savepoint and rollback to it
await db.transaction().execute(async (trx) => {
  await trx.insertInto("user").values({ email: "a@test.com" }).execute();
  
  // Create savepoint
  const savepoint = await trx.savepoint("before_risky_op");
  
  try {
    await trx.insertInto("user").values({ email: "duplicate@test.com" }).execute();
  } catch (e) {
    // Rollback to savepoint without canceling the entire transaction
    await savepoint.rollbackToSavepoint();
  }
  
  // First insert will be preserved
});
```

#### await using (Explicit Resource Management)

```typescript
// Automatic rollback when exiting scope without commit
await using trx = await db.startTransaction();

await trx.insertInto("user").values({ email: "test@example.com" }).execute();

// If commit() is not called, transaction will rollback automatically
await trx.commit();
```

#### Read-only Transactions

```typescript
// Read-only transaction (PostgreSQL optimizes these)
await db.transaction().setReadOnly(true).execute(async (trx) => {
  const users = await trx.selectFrom("user").selectAll().execute();
  // INSERT/UPDATE/DELETE will throw a database error
});
```

#### HandleEmptyInListsPlugin

```typescript
import { Kysely, HandleEmptyInListsPlugin } from "kysely";

// Plugin handles empty arrays in IN()
const db = new Kysely<DB>({
  dialect,
  plugins: [new HandleEmptyInListsPlugin()],
});

const ids: number[] = []; // Empty array

// Without plugin: SQL syntax error "IN ()"
// With plugin: automatically replaced with "1 = 0" (always false)
await db.selectFrom("user").where("id", "in", ids).execute();
```

#### Cross Join and Cross Join Lateral

```typescript
// Cross join
const result = await db
  .selectFrom("product")
  .crossJoin("category")
  .select(["product.name", "category.name as categoryName"])
  .execute();

// Cross join lateral (for correlated subqueries)
const result = await db
  .selectFrom("user as u")
  .crossJoinLateral(
    (eb) => eb
      .selectFrom("order")
      .select(["id", "total"])
      .whereRef("order.user_id", "=", "u.id")
      .orderBy("created_at", "desc")
      .limit(3)
      .as("recent_orders")
  )
  .selectAll()
  .execute();
```

#### DynamicModule.table() for Dynamic Tables

```typescript
import { DynamicModule } from "kysely";

const dynamic = new DynamicModule();

// Type-safe dynamic table reference
const tableName = "user" as "user" | "admin";
const users = await db
  .selectFrom(dynamic.table(tableName))
  .selectAll()
  .execute();
```

## Features from 0.27.x

### MERGE Queries (0.27.3+)

```typescript
// MERGE (upsert with full control)
await db
  .mergeInto("product")
  .using("product_updates", "product.sku", "product_updates.sku")
  .whenMatched()
  .thenUpdateSet({
    name: (eb) => eb.ref("product_updates.name"),
    price: (eb) => eb.ref("product_updates.price"),
  })
  .whenNotMatched()
  .thenInsertValues({
    sku: (eb) => eb.ref("product_updates.sku"),
    name: (eb) => eb.ref("product_updates.name"),
    price: (eb) => eb.ref("product_updates.price"),
  })
  .execute();
```

### Clear Methods (0.27.3+)

```typescript
// Clear specific query parts for dynamic query building
let query = db.selectFrom("user").select(["id", "email"]).where("role", "=", "admin");

// Clear and rebuild
query = query
  .clearSelect()
  .select(["id", "first_name", "last_name"])
  .clearWhere()
  .where("is_active", "=", true);

// Available: clearSelect(), clearWhere(), clearOrderBy(), clearLimit(), clearOffset(), clearGroupBy()
```

### eb.cast() Method (0.27.3+)

```typescript
// Type-safe casting
.select((eb) => [
  eb.cast<number>("price", "integer").as("priceInt"),
  eb.cast(eb.val("123"), "integer").as("numericValue"),
])
```

### WITHIN GROUP for Ordered Aggregates (0.27.4+)

```typescript
// PostgreSQL ordered-set aggregate functions
.select((eb) => [
  eb.fn("percentile_cont", [eb.lit(0.5)])
    .withinGroup((ob) => ob.orderBy("price", "asc"))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qwexs/kysely-postgres-skill](https://github.com/qwexs/kysely-postgres-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
