---
trigger: always_on
description: This document outlines the key aspects of using migrations with PostgreSQL in Payload CMS, based on the provided documentation. It focuses on how migrations are managed, their workflow, and best practices for PostgreSQL users.
---



# PostgreSQL Migrations with Payload CMS

This document outlines the key aspects of using migrations with PostgreSQL in Payload CMS, based on the provided documentation. It focuses on how migrations are managed, their workflow, and best practices for PostgreSQL users.

## Overview of Migrations in Payload CMS

Payload CMS provides a robust migration system to manage database schema changes. For PostgreSQL, migrations are critical because any modification to the Payload configuration (e.g., adding a new field or collection) requires corresponding updates to the database schema to avoid errors during data operations.

### Migration File Structure

A migration file in Payload CMS contains two primary functions:

- **Up Function**: Executes the changes to the database schema, such as creating tables or modifying columns.
- **Down Function**: Reverts changes made by the `up` function in case of a failure, ensuring the database remains consistent.

Example migration file for PostgreSQL:

```typescript
import { type MigrateUpArgs, sql } from '@payloadcms/db-postgres'

export async function up({ db, payload, req }: MigrateUpArgs): Promise<void> {
  // Perform schema changes, e.g., select data for transformation
  const { rows: posts } = await db.execute(sql`SELECT * from posts`);
  // Add logic to modify schema or data
}

export async function down({ db, payload, req }: MigrateUpArgs): Promise<void> {
  // Revert changes made in the up function
}
```

### Transaction Management

Payload CMS automatically wraps each migration in a transaction when using PostgreSQL. The `req` object must be passed to database operations (e.g., `payload.db.updateMany()`) to ensure changes occur within the transaction. If an error occurs, the transaction is aborted, preventing partial changes to the database.

Direct database operations are also supported within the transaction, as shown in the example above using the `db.execute` method with the `sql` template literal.

## Migration Commands

Payload CMS provides several commands to manage migrations, accessible via the `bun run payload` command, assuming an npm script is defined in `package.json` as follows:

```json
{
  "scripts": {
    "payload": "cross-env PAYLOAD_CONFIG_PATH=src/payload.config.ts payload"
  }
}
```

Key migration commands for PostgreSQL include:

- **Migrate**: Executes all unapplied migrations to update the database schema.
  ```bash
  bun run payload migrate
  ```

- **Create**: Generates a new migration file, optionally with a specified name. Payload uses Drizzle ORM to detect schema changes and generate the necessary SQL.
  ```bash
  bun run payload migrate:create optional-name-here
  ```
  Flags:
  - `--skip-empty`: Skips the prompt for creating a blank migration file when no schema changes are detected, useful for CI pipelines.
  - `--force-accept-warning`: Automatically accepts prompts to create a blank migration file.

- **Status**: Displays a table showing which migrations have been applied and which are pending.
  ```bash
  bun run payload migrate:status
  ```

- **Down**: Rolls back the last batch of migrations.
  ```bash
  bun run payload migrate:down
  ```

- **Refresh**: Rolls back all applied migrations and re-runs them.
  ```bash
  bun run payload migrate:refresh
  ```

- **Reset**: Rolls back all migrations.
  ```bash
  bun run payload migrate:reset
  ```

- **Fresh**: Drops all database entities and re-runs all migrations from scratch.
  ```bash
  bun run payload migrate:fresh
  ```

## PostgreSQL Migration Workflow

The workflow for managing migrations in PostgreSQL with Payload CMS is structured to support both development and production environments.

### 1. Local Development with Push Mode

In development, Payload CMS uses Drizzle ORM’s `push` mode to automatically synchronize schema changes to the local PostgreSQL database. This eliminates the need to run migrations manually during development, as changes to the Payload configuration are pushed directly to the database.

To disable `push` mode, set `push: false` in the PostgreSQL adapter configuration, but this may cause errors if the database schema lags behind the Payload configuration. Therefore, it is recommended to keep `push` mode enabled for local development and treat the local database as a sandbox.

### 2. Creating Migrations

Once development is complete or a feature is finalized, create a migration to capture schema changes:

- Run `bun run payload migrate:create` to generate a migration file.
- Payload compares the current schema with the Payload configuration and generates SQL to bridge the gap.
- The migration file is stored in the default directory (`./src/migrations`) or a custom directory specified via the `migrationDir` property in the database adapter configuration.

### 3. Running Migrations in Production

For production, migrations should be executed before building the application to ensure the database schema aligns with the Payload configuration. This is typically handled in a CI pipeline using a build script in `package.json`:

```json
{
  "scripts": {
    "dev": "next dev --turbo",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paideia-lms/Paideia](https://github.com/paideia-lms/Paideia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
