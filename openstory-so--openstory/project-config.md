---
trigger: always_on
description: Drizzle ORM patterns and database conventions
---


# Drizzle ORM Patterns

## Schema Definition

```typescript
import {
  pgTable,
  uuid,
  text,
  timestamp,
  boolean,
  jsonb,
  integer,
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const teams = pgTable('teams', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email'),
  isAnonymous: boolean('is_anonymous').default(true).notNull(),
  teamId: uuid('team_id').references(() => teams.id),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const sequences = pgTable('sequences', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  script: text('script'),
  teamId: uuid('team_id')
    .references(() => teams.id)
    .notNull(),
  createdBy: uuid('created_by')
    .references(() => users.id)
    .notNull(),
  styleId: uuid('style_id').references(() => styles.id),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const frames = pgTable('frames', {
  id: uuid('id').primaryKey().defaultRandom(),
  sequenceId: uuid('sequence_id')
    .references(() => sequences.id, { onDelete: 'cascade' })
    .notNull(),
  order: integer('order').notNull(),
  description: text('description').notNull(),
  thumbnailUrl: text('thumbnail_url'),
  status: text('status').notNull().default('pending'), // pending, processing, completed, failed
  error: text('error'),
  createdBy: uuid('created_by')
    .references(() => users.id)
    .notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const styles = pgTable('styles', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  description: text('description'),
  teamId: uuid('team_id')
    .references(() => teams.id)
    .notNull(),
  styleData: jsonb('style_data').notNull(), // Style Stack JSON
  createdBy: uuid('created_by')
    .references(() => users.id)
    .notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// Relations
export const teamsRelations = relations(teams, ({ many }) => ({
  users: many(users),
  sequences: many(sequences),
  styles: many(styles),
}));

export const usersRelations = relations(users, ({ one, many }) => ({
  team: one(teams, {
    fields: [users.teamId],
    references: [teams.id],
  }),
  sequences: many(sequences),
  frames: many(frames),
  styles: many(styles),
}));

export const sequencesRelations = relations(sequences, ({ one, many }) => ({
  team: one(teams, {
    fields: [sequences.teamId],
    references: [teams.id],
  }),
  frames: many(frames),
  style: one(styles, {
    fields: [sequences.styleId],
    references: [styles.id],
  }),
  creator: one(users, {
    fields: [sequences.createdBy],
    references: [users.id],
  }),
}));

export const framesRelations = relations(frames, ({ one }) => ({
  sequence: one(sequences, {
    fields: [frames.sequenceId],
    references: [sequences.id],
  }),
  creator: one(users, {
    fields: [frames.createdBy],
    references: [users.id],
  }),
}));

export const stylesRelations = relations(styles, ({ one, many }) => ({
  team: one(teams, {
    fields: [styles.teamId],
    references: [teams.id],
  }),
  creator: one(users, {
    fields: [styles.createdBy],
    references: [users.id],
  }),
  sequences: many(sequences),
}));
```

## Database Client Setup

```typescript
// apps/backend/src/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.POSTGRES_URL;

if (!connectionString) {
  throw new Error('POSTGRES_URL environment variable is not set');
}

// Create PostgreSQL connection
const queryClient = postgres(connectionString);

// Create Drizzle instance
export const db = drizzle(queryClient, { schema });

// Type exports
export type Database = typeof db;
```

## Query Patterns

```typescript
import { db } from './index';
import { sequences, frames } from './schema';
import { eq, and, desc, asc, count } from 'drizzle-orm';

// Simple select with relations
export async function getSequenceWithFrames(sequenceId: string) {
  return db.query.sequences.findFirst({
    where: eq(sequences.id, sequenceId),
    with: {
      frames: {
        orderBy: [asc(frames.order)],
      },
      style: true,
      creator: {
        columns: {
          id: true,
          email: true,
          isAnonymous: true,
        },
      },
    },
  });
}

// List with pagination
export async function listSequences(
  teamId: string,
  page: number = 1,
  pageSize: number = 20
) {
  const offset = (page - 1) * pageSize;

  const [items, totalCount] = await Promise.all([
    db.query.sequences.findMany({
      where: eq(sequences.teamId, teamId),
      limit: pageSize,
      offset,
      orderBy: [desc(sequences.createdAt)],
      with: {
        frames: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
