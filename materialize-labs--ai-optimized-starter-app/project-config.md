---
trigger: always_on
description: Follow these rules when working on the backend.
---

# Backend Architecture Guidelines

This document outlines our backend architecture and coding standards. Our backend stack includes:

- **Database**: PostgreSQL via Supabase
- **ORM**: Drizzle for type-safe database access
- **API Layer**: Next.js Server Actions
- **Infrastructure**: Serverless via Vercel

## Core Principles

1. **Type Safety**: Maintain full type safety between database and application code
2. **Scalability**: Design for scale from the beginning
3. **Maintainability**: Follow consistent patterns across the codebase
4. **Security**: Validate all inputs and handle user permissions properly
5. **Performance**: Optimize database queries and API responses

## Database Schema Design

### Directory Structure

```
db/
├── db.ts                # Main database configuration
├── index.ts             # Public exports
├── migrations/          # Database migrations
└── schema/              # Database schema definitions
    ├── [entity].ts  # Individual entity schemas
```

### Schema Definition Standards

#### Naming Conventions

- Use kebab-case for files: `contacts.ts`
- Use camelCase for table and column names in code
- Use snake_case for the actual database column names
- Export types with standardized prefixes: `Select[Entity]` and `Insert[Entity]`

#### Required Fields

All tables should include these standard fields:

```typescript
{
  // Always use UUID for primary keys
  id: uuid("id").defaultRandom().primaryKey(),
  
  // User association (when applicable)
  userId: text("user_id").notNull(),
  
  // Timestamps
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true })
    .defaultNow()
    .notNull()
    .$onUpdate(() => new Date())
}
```

#### Using Enums

For fields with a fixed set of values, use PostgreSQL enums:

```typescript
import { pgEnum } from "drizzle-orm/pg-core"

// Define the enum
export const statusEnum = pgEnum("status", ["active", "pending", "archived"])

// Use it in your schema
status: statusEnum("status").notNull().default("pending")
```

#### Relationships

Always define explicit relationships between tables and include appropriate cascade behavior:

```typescript
// One-to-many relationship example
projectId: uuid("project_id")
  .references(() => projectsTable.id, { onDelete: "cascade" })
  .notNull()
```

### Schema Example

```typescript
// db/schema/contacts.ts
import { pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core"

export const contactsTable = pgTable("contacts", {
  id: uuid("id").defaultRandom().primaryKey(),
  userId: text("user_id").notNull(),
  name: text("name").notNull(),
  email: text("email"),
  phone: text("phone"),
  notes: text("notes"),
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { withTimezone: true })
    .defaultNow()
    .notNull()
    .$onUpdate(() => new Date())
})

export type InsertContact = typeof contactsTable.$inferInsert
export type SelectContact = typeof contactsTable.$inferSelect
```

Make sure to export your schema from the index file:

```typescript
// db/schema/index.ts
export * from "./contacts"
// ... other schema exports
```

And register it in your database configuration:

```typescript
// db/db.ts
import { contactsTable } from "@/db/schema"

export const schema = {
  contacts: contactsTable,
  // ... other tables
}
```

## Server Actions

Server Actions are our primary method for exposing backend functionality. They provide type-safe, secure APIs for frontend components.

### Organization Pattern

```
actions/
├── auth/                # Authentication-related actions
├── db/                  # Database operations
│   ├── [entity].ts
└── utils/               # Utility actions
```

### Action Implementation Guidelines

1. **Standardized Return Type**: Use the `ActionState<T>` pattern for consistency

```typescript
export type ActionState<T> =
  | { isSuccess: true; message: string; data: T }
  | { isSuccess: false; message: string; data?: never }
```

2. **Naming Convention**: All action functions should end with `Action` suffix

3. **Input Validation**: Validate inputs before database operations

4. **Error Handling**: Use try/catch blocks and provide meaningful error messages

5. **Organization**: Group actions by entity and order methods by CRUD operations

### Server Action Example

```typescript
// actions/db/contacts.ts
'use server'

import { eq } from 'drizzle-orm'

import { db } from '@/db/db'
import { contactsTable, InsertContact, SelectContact } from '@/db/schema/contacts'
import { ActionState } from '@/types/server-action'

export async function createContact(contact: InsertContact): Promise<ActionState<SelectContact>> {
  try {
    const [newContact] = await db.insert(contactsTable).values(contact).returning()

    return {
      isSuccess: true,
      message: 'Contact created successfully',
      data: newContact,
    }
  } catch (error) {
    console.error('Error creating contact:', error)

    return { isSuccess: false, message: 'Failed to create contact' }
  }
}

export async function getContacts(userId: string): Promise<ActionState<SelectContact[]>> {
  try {
    const contacts = await db.query.contacts.findMany({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
