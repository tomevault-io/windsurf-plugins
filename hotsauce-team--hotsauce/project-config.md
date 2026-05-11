---
trigger: always_on
description: Guidelines for AI coding assistants working on this project.
---

# AGENTS.md

Guidelines for AI coding assistants working on this project.

## ⛔ CRITICAL RULES

**Read these first. Violating these will break the project or CI.**

1. **NO `--allow-*` FLAGS** — Run tests with `deno task test` or `deno test -P`. Never pass `--allow-read`, `--allow-env`, `--allow-net`, etc. Permissions are configured in `deno.jsonc`.

2. **STRICT DEPENDENCY POLICY** — Only pre-approved packages are allowed in production (see Dependencies section). Do not suggest adding packages without meeting all approval criteria.

3. **NO `npm`/`yarn`/`pnpm`** — This is a Deno project. Use `deno` commands only.

4. **NO `Deno.*` IN PACKAGES** — Code must be runtime-agnostic. Use Web Standard APIs only (`Request`, `Response`, `crypto`, `fetch`).

---

## Core Constraints

### Dependencies

- **Approved production dependencies:**
  - `drizzle-orm`
  - `postgres` (postgres.js driver)
  - `zod`
  - `drizzle-zod`
  - `@std/media-types` (Deno standard library — MIME type validation for file uploads)
- All approved packages have **zero transitive dependencies** — keep it that way
- Do NOT suggest adding packages without meeting **all** approval criteria:
  1. Zero transitive dependencies
  2. Pure data or pure functions (no runtime-specific APIs)
  3. From a trusted source (Drizzle team or Deno standard library)
  4. Published on JSR with integrity hashes
  5. Could we realistically maintain our own version? (If no, dependency is justified)

### Optional Peer Dependencies

Some features require external packages that users install only if needed:

- **qrcode-generator** — QR code generation for 2FA setup
  - Only required if using built-in password auth with 2FA enabled
  - Not needed for external auth providers (OAuth, SAML, etc.)
  - **Supply chain risk:** This is an npm package outside our control
  - **Recommendation:** Pin to a specific version you have audited
  - Install with: `npm install qrcode-generator@2.0.4`

### Dev Dependencies (testing only)

- `@electric-sql/pglite` — in-memory Postgres for tests
- `sql.js` — in-memory SQLite for tests
- `@std/assert` — Deno standard library assertions
- Dev dependencies are OK since they don't ship to users

### Runtime Compatibility

- All packages must be **runtime-agnostic**
- No `Deno.*` or Node-specific APIs — use Web Standard APIs only
- Handlers use Web Standard `Request`/`Response` (works in Deno, Node 20+, Bun, Workers)
- BYOS (Bring Your Own Server) architecture — users wire handlers to their server

## Package Boundaries

| Package | Purpose                                         | Runtime APIs         | DB-Specific Code | DB-Specific Tests  |
| ------- | ----------------------------------------------- | -------------------- | ---------------- | ------------------ |
| `core`  | Schema introspection, field mapping, validation | ❌ None              | ❌ Generic only  | ✅ PGlite + sql.js |
| `ui`    | HTML generation, form rendering                 | ❌ None              | ❌ Generic only  | ❌ None            |
| `cms`   | CRUD route handlers (Request → Response)        | ❌ Web Standard only | ❌ Generic only  | ✅ PGlite + sql.js |

## Database Guidelines

### Database-Agnostic Design

- Core schema introspection must work with **any** Drizzle schema (pg, mysql, sqlite)
- Use Drizzle's generic types in core, not `drizzle-orm/pg-core` directly
- Database-specific features (arrays, enums, JSON) should:
  - Live in database-specific modules or be feature-detected
  - Degrade gracefully when not available

### Drizzle ORM Helper Functions

- **Use exported helper functions** instead of direct symbol/property access
- Drizzle exports utilities like `getTableName`, `getTableColumns`, `isTable`
- These provide type-safe access without needing `as unknown as` casts

```typescript
// Good: use Drizzle's helpers
import { getTableColumns, getTableName, Table } from 'drizzle-orm';

const name = getTableName(table); // type-safe
const cols = getTableColumns(table); // returns typed columns object

// Bad: direct symbol access
const TABLE_NAME = Symbol.for('drizzle:Name');
const name = (table as any)[TABLE_NAME]; // loses type safety
```

- For properties without helpers (e.g., foreign keys), use symbol access with appropriate casts
- Check drizzle-orm's exports before adding custom symbol lookups

### Drizzle Compatibility Testing

The CMS extends Drizzle column builders with a `$cms()` method (e.g., to mark JSON columns as file fields). This requires patching Drizzle's `PgColumnBuilder`, `SQLiteColumnBuilder`, and `MySqlColumnBuilder` prototypes and relies on the internal `config` property flowing from builder to built column.

**Why we test Drizzle internals:**

- The `config` property is `protected`, not part of Drizzle's public API
- Class names like `PgColumnBuilder` could be renamed or restructured
- The config flow from builder → column is undocumented behavior

**What the tests verify:**

- Column builder classes exist and are accessible
- The `config` property exists and accepts custom properties
- Custom properties survive method chaining (`.notNull().default()`)
- Custom properties flow from builder to the built column
- Works across Postgres, SQLite, and MySQL

**Files:**

- `drizzle-compat.json` — Version matrix (tested, minimum, known_broken)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hotsauce-team/hotsauce](https://github.com/hotsauce-team/hotsauce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
