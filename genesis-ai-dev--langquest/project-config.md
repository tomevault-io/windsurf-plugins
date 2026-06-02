---
trigger: always_on
description: Guidelines for modifying Drizzle schema and creating migrations. Auto-attached when editing Supabase migrations, Drizzle schema files, or db/constants.ts.
---


# Database: Drizzle Schema Migration Process

## ⛔ Hard rule: version bumps come in pairs

**`APP_SCHEMA_VERSION` and `get_schema_info()` change together, or neither changes. There is no third option.**

Run this check on every migration diff:

| `APP_SCHEMA_VERSION` bumped? | Migration contains `create or replace function public.get_schema_info()`? | Verdict |
|---|---|---|
| ❌ No | ❌ No | ✅ Server-only — Path A |
| ✅ Yes | ✅ Yes (matching version) | ✅ Client schema change — Path B |
| ❌ No | ✅ Yes | 🛑 Broken — remove the `get_schema_info()` block |
| ✅ Yes | ❌ No | 🛑 Broken — add the `get_schema_info()` bump |

**Default = Path A.** If you cannot point to a specific client code change (a Drizzle column the app reads/writes, a sync-rules change clients must satisfy, a renamed table the client queries), do not touch `APP_SCHEMA_VERSION` or `get_schema_info()`. Incorrectly bumping either blocks sync for users on current app builds.

> **Real incidents this prevents.** `20260529120000_rename_project_language_suggestion_to_languoid.sql` shipped a `get_schema_info()` bump to `2.5 / min 2.4` while `APP_SCHEMA_VERSION` stayed at `2.3` — blocked sync for every user on the current build and required `20260530120000_revert_schema_info_to_2_3.sql` to undo. `20260520210000_invite_bounce_type_and_reason.sql` made the same mistake (server-only `bounce_type`/`bounce_reason` columns, bumped to `2.4` anyway).

## Decision: Path A or Path B?

| Change type | Supabase migration | `APP_SCHEMA_VERSION` | `get_schema_info()` | Drizzle schema | Local `db/migrations/` |
|---|---|---|---|---|---|
| Server-only nullable column | ✅ | ❌ | ❌ | ❌ | ❌ |
| RLS / triggers / functions / indexes | ✅ | ❌ | ❌ | ❌ | ❌ |
| Client reads/writes new synced column | ✅ | minor bump | ✅ | ✅ | maybe |
| Destructive client column change | ✅ | major bump | ✅ | ✅ | ✅ |
| Server RPC transform for legacy uploads | ✅ | with client | ✅ | maybe | ❌ |

**Path A — server-only:** Postgres changes the client doesn't see. Examples: nullable columns the app never reads, RLS, triggers, indexes, server functions, backfills, invite bounce-tracking columns. Only output: a Supabase SQL migration. Do **not** edit `db/constants.ts`, the Drizzle schemas, or `db/migrations/`.

**Path B — client schema change:** App code or the client sync contract changes. Examples: client reads/writes a new column, required field, rename/drop a column the client uses, sync-rules change old clients can't satisfy. Coordinate steps 1–6 below.

## PowerSync is schemaless

PowerSync stores synced data as schemaless JSON; the client Drizzle schema is just a view on top.

> "Updating this client-side schema is immediate when the new version of the app runs, with no client-side migrations required." — [PowerSync: Implementing Schema Changes](https://docs.powersync.com/usage/lifecycle-maintenance/implementing-schema-changes)

Practical consequences:

- **Adding a nullable column** to a synced table does not need a local SQLite migration. Existing records read `undefined`, equivalent to `NULL`.
- **Local migrations are needed** only when client code actively reads/writes a column with non-null semantics, when removing/renaming a column the client uses, or when existing local data needs transformation/backfill.
- Both synced tables and `*_local` tables use the same JSON-storage architecture — the difference is sync scope, not storage mechanism.

## Path A: Server-only Supabase migration

```sql
-- Migration: Add server_only_flag to invite (server-side delivery tracking)
-- NO schema version bump — server-only, nullable, not in client Drizzle schema

alter table public.invite
  add column if not exists server_only_flag text;

comment on column public.invite.server_only_flag is 'Used by Resend webhook; not synced to client.';
```

Do **not** include `create or replace function public.get_schema_info()` in this migration.

## Path B: Step-by-step (client schema change)

### 1. Modify shared schema definitions

Edit `db/drizzleSchemaColumns.ts` — its functions are reused by both `drizzleSchema.ts` (synced tables) and `drizzleSchemaLocal.ts` (`*_local` tables).

```typescript
export function createAssetTable(source: TableSource, refs: {...}) {
  return tableCreator(source)('asset', {
    ...getBaseColumns(source),
    new_field: text(),
  });
}
```

### 2. Bump `APP_SCHEMA_VERSION` + update `get_schema_info()`

These two move together (see hard rule). Format: `MAJOR.MINOR`.

- **Minor bump** (`2.0 → 2.1`): additive client changes (new columns/tables the app uses).
- **Major bump** (`2.0 → 3.0`): destructive client changes (removed/renamed columns, type changes, dropped tables).

```typescript
// db/constants.ts
export const APP_SCHEMA_VERSION = '2.1';
```

```sql
-- in your Supabase migration
create or replace function public.get_schema_info()
returns jsonb
language sql
security invoker
set search_path = public
as $$
  select jsonb_build_object(
    'schema_version', '2.1',
    'min_required_schema_version', '2.0',
    'notes', 'Clients must be at least 2.0 to sync.'
  );
$$;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genesis-ai-dev/langquest](https://github.com/genesis-ai-dev/langquest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
