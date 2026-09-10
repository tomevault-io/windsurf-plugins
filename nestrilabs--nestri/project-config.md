---
trigger: always_on
description: Every domain module lives in `packages/core/src/<parent>/` as either a top-level namespace or a nested sub-module:
---

# `packages/core` — domain modules

## Structure

Every domain module lives in `packages/core/src/<parent>/` as either a top-level namespace or a nested sub-module:

```
src/<parent>/
  ├── <parent>.sql.ts        # (optional) Drizzle table for the parent entity
  ├── index.ts               # Parent namespace (e.g. User, Game, Team)
  ├── <child>.sql.ts         # Sub-module table (e.g. fingerprint.sql.ts)
  └── <child>.ts             # Sub-module namespace (e.g. export namespace Fingerprint)
```

### Sub-modules nested under parents

| File                    | Namespace       | Why                                   |
| ----------------------- | --------------- | ------------------------------------- |
| `user/linked-account.*` | `LinkedAccount` | A user's OAuth/gaming identities      |
| `user/fingerprint.*`    | `Fingerprint`   | SSH key fingerprints                  |
| `game/download.*`      | `GameDownload`  | Per-host game depot downloads         |
| `user/library.*`        | `Library`       | User's owned games with playtime      |
| `team/member.*`         | `Member`        | Team membership with role             |
| `game/depot.*`          | `Depot`         | Platform-specific game content depots |
| `steam/enrolment.*`     | `Enrolment`     | Which host holds a Steam token for whom |

Existing top-level modules: `user/`, `team/`, `game/`, `pairing-code/`, `steam/`, `auth/`, `db/`.

A parent may own no table of its own and still have sub-modules that do: `steam/index.ts` is reusable `fn()` functions with no `.sql.ts` beside it, while `steam/enrolment.*` is a full pair.

## Pattern: `.sql.ts` (Drizzle Table)

```ts
// src/<module>/<module>.sql.ts
import { pgTable, text, boolean, jsonb, uniqueIndex, index, pgEnum } from 'drizzle-orm/pg-core';

import { id, timestamps, ulid } from '../db/types.js';

// Enum (only if needed — co-located with its table)
export const SomeEnum = pgEnum('some_enum', ['a', 'b']);

// FK imports — use the sql.ts files, never the index.ts (avoids circular deps)
import { UserTable } from '../user/user.sql.js';

export const SomeTable = pgTable(
	'some_table',
	{
		...id, // char(30) PK, prefix: som_
		...timestamps, // time_created, time_updated, time_deleted (all utc)

		// FK column — always use ulid() + .references()
		userId: ulid('user_id')
			.notNull()
			.references(() => UserTable.id, { onDelete: 'cascade' }),

		// Scalar columns
		name: text('name').notNull(),
		email: text('email'), // nullable = omit .notNull()
		flag: boolean('flag').notNull().default(false),
		metadata: jsonb('metadata').$type<{}>(), // JSON blob

		// Enum column
		provider: SomeEnum('provider').notNull()
	},
	(t) => [
		uniqueIndex('some_table_provider_unique').on(t.provider, t.providerAccountId),
		index('some_table_sync_idx')
			.on(t.userId)
			.where(sql`${t.localValue} is distinct from ${t.remoteValue}`),
		index('some_table_user_idx').on(t.userId)
	]
);
```

### DB types (`src/db/types.ts`)

| Helper       | Output                                                          |
| ------------ | --------------------------------------------------------------- |
| `ulid(name)` | `char(30)` — for PKs and FKs                                    |
| `id`         | `{ id: ulid('id').primaryKey().notNull() }` — spread as `...id` |
| `utc(name)`  | `timestamp with time zone`                                      |
| `timestamps` | `{ timeCreated, timeUpdated (auto), timeDeleted }`              |

### Naming conventions

- Table name: `snake_case` (e.g. `linked_account`, `team_member`)
- Column name: `snake_case` (e.g. `user_id`, `provider_account_id`, `time_created`)
- TypeScript field names: `camelCase` matching the column (drizzle maps them)
- Index names: `{table}_{column(s)}_unique` / `{table}_{column}_idx`

---

## Pattern: `index.ts` (Domain Namespace)

```ts
// src/<module>/index.ts
import { eq, and, isNull, sql } from 'drizzle-orm';
import z from 'zod';

import { Database } from '../db/index.js';
import { Examples } from '../examples.js';
import { fn } from '../fn.js';
import { SomeTable, SomeEnum } from './<module>.sql.js';

export namespace SomeModule {
  // ── Info schema ─────────────────────────────────────────────────────
  // Single source of truth for the entity shape.
  // Every field typed here; .meta() adds OpenAPI metadata.
  // When a field changes here, TypeScript catches every usage.
  export const Info = z
    .object({
      id: z.string().meta({
        description: '…',
        example: Examples.SomeModule.id,
      }),
      // For enum fields, use z.enum(SomeEnum.enumValues) to stay in sync:
      provider: z.enum(SomeEnum.enumValues).meta({ … }),
      // Nullable + optional for JSON-blob / optional fields:
      metadata: z.record(z.string(), z.unknown()).nullable().optional().meta({ … }),
    })
    .meta({
      ref: 'SomeModule',
      description: '…',
      example: Examples.SomeModule,
    });

  export type Info = z.infer<typeof Info>;

  // ── create ───────────────────────────────────────────────────────────
  // Use Info.pick({…}) for the schema — keeps fields in sync with Info.
  // Input is the parsed object.
  // Use Database.use() for single-operation writes.
  export const create = fn(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nestrilabs/nestri](https://github.com/nestrilabs/nestri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
