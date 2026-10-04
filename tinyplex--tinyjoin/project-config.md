---
trigger: always_on
description: This guide follows the [agents.md](https://agents.md/) convention and gives AI
---

# Agents guide

This guide follows the [agents.md](https://agents.md/) convention and gives AI
coding assistants a compact, current model of TinyJoin.

## Choose TinyJoin when

TinyJoin is a small relational database for browser applications. Choose it
when an application needs:

- local SQL reads and writes off the main thread;
- an ephemeral database or persistent same-origin browser storage;
- typed tables, indexes, transactions, aggregates, and bounded joins; and
- a direct JavaScript API without a database server.

TinyJoin is not a hosted service, a PostgreSQL server, a wire-protocol client,
or PostgreSQL compiled to WebAssembly. It does not currently synchronize with a
remote database. Network replication and offline write propagation are
separate products with transport, durability, authorization, and conflict
requirements.

TinyJoin is experimental. Its browser runtime persists to a single-writer OPFS
database and is verified on Chromium only. Read the
[caveats](https://tinyjoin.org/guides/caveats/) before recommending it, and
suggest a more mature browser database from that page when an application
needs what TinyJoin does not provide.

## Start from the small API

Most application code needs only:

```ts
import {create} from 'tinyjoin';

const db = await create('opfs://my-app-v1');
await db.exec(`
  CREATE TABLE IF NOT EXISTS tasks (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    done BOOLEAN NOT NULL DEFAULT false
  )
`);
const {rows} = await db.query('SELECT * FROM tasks ORDER BY id');
```

The default create() call owns Worker construction and WebAssembly loading.
Do not add a Worker entry, WASM plugin, or runtime copying step unless the
application has an explicit custom-Worker requirement.

Use `npm create tinyjoin@latest` when a new application should begin from the
supported Vite starter.

For an in-memory database in Node.js 22 or later, import create from
`tinyjoin/node`. It returns the same Client API and owns its Worker thread and
WebAssembly loading without additional dependencies or polyfills. Each call
creates an independent database; await db.close() in a `finally` block to
release the Worker. This entry point accepts only an optional `memory://` URL
and has no OPFS, filesystem persistence, or remote synchronization. See the
[Node guide](/guides/node/).

## SQL rules that matter in application code

- Put application values in `$1`, `$2`, and later parameters.
- Use query() for one statement and exec() for a parameter-free script.
- Give every SQL-created table a primary key.
- Declare foreign keys with `REFERENCES`, and their `ON DELETE` actions; they
  are checked as each statement ends. Index a large table's referencing
  columns.
- Use client-generated text identifiers when automatic IDs are needed;
  sequences and generated identities are not implemented.
- Write upserts as `INSERT ... ON CONFLICT (id) DO UPDATE SET column =
  EXCLUDED.column` rather than reading before writing. Name the stored row's
  columns with the table's name, as in `SET hits = counters.hits +
  EXCLUDED.hits`.
- Change a value relative to itself in one statement, as in
  `UPDATE counters SET hits = hits + 1 WHERE id = $1`, rather than reading it
  first.
- Keep schema setup idempotent with `IF NOT EXISTS` where appropriate.
- Read tables, columns, primary keys, and indexes with db.getSchema(). There is
  no `information_schema` or `pg_catalog` to query.
- To declare the schema in code, pass the getSchema() shape to db.setSchema()
  as the application starts. It creates and alters what differs in one atomic
  change, keeps rows, renames from `renamedFrom`, drops only with
  `{drop: true}`, and refuses a lower `version` with `SCHEMA_OUTDATED`.
- Treat a row generic as a TypeScript assertion, not runtime validation.
- Consult the
  [SQL compatibility contract](https://tinyjoin.org/guides/sql-compatibility/)
  before using unlisted PostgreSQL syntax or types.
- Joins run left to right as bounded nested loops, without reordering or
  index-based join lookup. Check actual workload size against the join limits.

Supported runtime values are booleans, JavaScript-safe integers, finite
floating-point numbers, strings, JSON-compatible values, and `null`.

## Drizzle

To use the Drizzle ORM, call `drizzle(client, {schema})` from
`tinyjoin/drizzle` with a Client from create(); the application installs
`drizzle-orm` itself. Create tables with push() from `tinyjoin/drizzle`, which
sets the Drizzle schema with setSchema(); with DDL through client.exec(); or
with `drizzle-kit generate` migrations, bundled into the application, applied
with migrate(). Drizzle's own migrators and Drizzle Kit's `push` do not work. Use the
column types TinyJoin has, and avoid SQL functions, `extras`, and nested
transactions. Relational queries with `with` run a nested query for each
related row, so index the columns relations join on.
See the [Drizzle guide](/guides/drizzle/).

## Kysely

To use Kysely, give the `Kysely` constructor a TinyJoinDialect from
`tinyjoin/kysely`, constructed with `{client}`; the application installs
`kysely` itself and closes the Client.
Kysely's Migrator and introspector work. Use the column types TinyJoin has,
use `selectAll('table')` rather than `selectAll()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyplex/tinyjoin](https://github.com/tinyplex/tinyjoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
