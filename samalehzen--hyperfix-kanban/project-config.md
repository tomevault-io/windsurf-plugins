---
trigger: always_on
description: Database schema conventions and patterns using Drizzle ORM and PostgreSQL
---

# Database Schema Guidelines

HyperFix uses Drizzle ORM with PostgreSQL. This document outlines schema conventions and best practices.

## Schema Definition

All schemas are defined in `apps/api/src/database/schema.ts`:

```typescript
import { pgTable, text, timestamp, boolean, integer, index } from "drizzle-orm/pg-core";
import { createId } from "@paralleldrive/cuid2";

export const taskTable = pgTable(
  "task",
  {
    id: text("id")
      .$defaultFn(() => createId())
      .primaryKey(),
    projectId: text("project_id")
      .notNull()
      .references(() => projectTable.id, {
        onDelete: "cascade",
        onUpdate: "cascade",
      }),
    title: text("title").notNull(),
    description: text("description"),
    status: text("status").notNull().default("to-do"),
    createdAt: timestamp("created_at", { mode: "date" }).defaultNow().notNull(),
    updatedAt: timestamp("updated_at", { mode: "date" })
      .defaultNow()
      .$onUpdate(() => new Date())
      .notNull(),
  },
  (table) => [
    index("task_projectId_idx").on(table.projectId),
  ],
);
```

## Naming Conventions

1. **Table names**: Use singular, lowercase with underscores: `task`, `workspace_user`
2. **Column names**: Use snake_case: `project_id`, `created_at`, `updated_at`
3. **Index names**: Format: `{table}_{column}_idx`
4. **Foreign key columns**: End with `_id`: `project_id`, `user_id`

```typescript
// Good: Consistent naming
export const workspaceUserTable = pgTable("workspace_user", {
  id: text("id").primaryKey(),
  workspaceId: text("workspace_id").notNull(),
  userId: text("user_id").notNull(),
  role: text("role").notNull(),
});

// Bad: Inconsistent naming
export const workspaceUserTable = pgTable("WorkspaceUsers", {
  ID: text("ID").primaryKey(),
  workspace: text("workspace").notNull(),
  user: text("user").notNull(),
});
```

## Required Fields

Every table should include:

1. **Primary Key**: `id` field using CUID2
2. **Timestamps**: `created_at` and `updated_at` with proper defaults
3. **Foreign Keys**: Proper references with cascade options

```typescript
export const exampleTable = pgTable("example", {
  // Required: Primary key with CUID2
  id: text("id")
    .$defaultFn(() => createId())
    .primaryKey(),

  // Required: Timestamps
  createdAt: timestamp("created_at", { mode: "date" })
    .defaultNow()
    .notNull(),
  updatedAt: timestamp("updated_at", { mode: "date" })
    .defaultNow()
    .$onUpdate(() => new Date())
    .notNull(),

  // Foreign keys with cascade
  projectId: text("project_id")
    .notNull()
    .references(() => projectTable.id, {
      onDelete: "cascade",
      onUpdate: "cascade",
    }),
});
```

## ID Generation

Always use CUID2 for primary keys:

```typescript
import { createId } from "@paralleldrive/cuid2";

id: text("id")
  .$defaultFn(() => createId())
  .primaryKey(),
```

## Timestamps

Use consistent timestamp patterns:

```typescript
// Created timestamp (never changes)
createdAt: timestamp("created_at", { mode: "date" })
  .defaultNow()
  .notNull(),

// Updated timestamp (auto-updates on change)
updatedAt: timestamp("updated_at", { mode: "date" })
  .defaultNow()
  .$onUpdate(() => new Date())
  .notNull(),

// Optional timestamp (e.g., deleted_at)
deletedAt: timestamp("deleted_at", { mode: "date" }),
```

## Foreign Keys

Always specify cascade behavior:

```typescript
// Good: Explicit cascade behavior
userId: text("user_id")
  .notNull()
  .references(() => userTable.id, {
    onDelete: "cascade",
    onUpdate: "cascade",
  }),

// Bad: Missing cascade options
userId: text("user_id").references(() => userTable.id),
```

### Cascade Options

- **`onDelete: "cascade"`**: Delete related records when parent is deleted
- **`onUpdate: "cascade"`**: Update foreign key when parent ID changes
- **`onDelete: "set null"`**: Set foreign key to null when parent is deleted (use nullable column)

## Indexes

Add indexes for frequently queried columns:

```typescript
export const taskTable = pgTable(
  "task",
  {
    // ... columns
    projectId: text("project_id").notNull(),
    userId: text("user_id"),
    status: text("status").notNull(),
  },
  (table) => [
    index("task_projectId_idx").on(table.projectId),
    index("task_userId_idx").on(table.userId),
    index("task_status_idx").on(table.status),
  ],
);
```

### When to Add Indexes

- Foreign key columns (always)
- Columns used in WHERE clauses frequently
- Columns used for sorting/ordering
- Composite indexes for multi-column queries

## Relations

Define relations in `apps/api/src/database/relations.ts`:

```typescript
import { relations } from "drizzle-orm";
import { taskTable } from "./schema";
import { projectTable } from "./schema";

export const taskTableRelations = relations(taskTable, ({ one, many }) => ({
  project: one(projectTable, {
    fields: [taskTable.projectId],
    references: [projectTable.id],
  }),
  labels: many(labelTable),
}));
```

## Data Types

Use appropriate PostgreSQL types:

```typescript
// Text (varchar)
title: text("title").notNull(),

// Optional text
description: text("description"),

// Boolean
isActive: boolean("is_active").default(true).notNull(),

// Integer
position: integer("position").default(0),

// Timestamp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamalehZen/HyperFix-Kanban](https://github.com/SamalehZen/HyperFix-Kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
