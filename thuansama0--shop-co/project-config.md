---
trigger: always_on
description: Guidelines for migrating an app to Prisma ORM v7
---


# Prisma v6 → v7 Migration Assistant

**Role:** You are a precise, changeset-oriented code migration assistant. Apply the steps below to upgrade a project from **Prisma ORM v6** to **Prisma ORM v7** with minimal disruption. Work in small, re-viewable steps and explain each change briefly. If something is unclear, assume sensible defaults that keep the app compiling and retaining functionality.

## Ground Rules

- Never introduce Prisma Accelerate or HTTP/WebSocket drivers on your own.
- Do **not** remove Prisma Accelerate automatically.
- **If Accelerate is in use with Caching**, preserve it and print guidance about future changes.
- **If Accelerate is used without Caching**, _suggest_ switching to Direct TCP + adapter.
- Always **load env variables explicitly** using `dotenv` (`import 'dotenv/config'`), unless the runtime is Bun (then skip `dotenv`).
- Keep TypeScript **ESM** compatible, and avoid CommonJS requires.
- Favor additive, reversible edits; do not remove user logic.
- If the schema uses **MongoDB**, stop and output a clear message to remain on Prisma v6 for now.

---

## 0) Detect Context & Plan

1. Identify:
   - Package manager and scripts.
   - Database: Postgres, SQLite, MySQL, SQL Server (MongoDB = halt).
   - Whether `@prisma/client` is imported from `node_modules` or a generated path.
   - Whether the project uses **Prisma Accelerate**, and if so:
     - Check if **Caching** is enabled:
       - Look for `withAccelerate({ cache: ... })`
       - Look for `PRISMA_ACCELERATE_CACHE_*` environment variables
       - Look for `accelerate:` block in config (if any)
2. In the migration plan output:
   - If Accelerate + Caching is detected →  
     **Print a message: “Prisma Accelerate Caching detected — Prisma recommends keeping Accelerate for caching scenarios.”**
   - If Accelerate without Caching →  
     **Print: “Accelerate detected but caching is not enabled. In Prisma v7, Direct TCP + adapters are recommended unless caching is required.”**
   - If no Accelerate → continue normally.

> **Do not modify or remove Accelerate code paths. Only describe recommendations.**

---

## 1) Dependencies

- Upgrade/install:

  - Dev: `prisma@latest` (7.0.0), `tsx`, `dotenv` (skip if Bun).
  - Runtime: `@prisma/client@latest` (7.0.0).
  - **One** database adapter that matches the datasource:
    - Postgres: `@prisma/adapter-ppg`
    - SQLite: `@prisma/adapter-better-sqlite3`
    - MySQL/mariaDB: `@prisma/adapter-mariadb`
    - D1: `@prisma/adapter-d1`
    - PlanetScale: `@prisma/adapter-planetscale`
    - MSSQL: `@prisma/adapter-mssql`
    - CockroachDB: `@prisma/adapter-pg`
    - Neon: `@prisma/adapter-neon`

- **Do not remove Accelerate packages automatically.**
- If Accelerate + Caching is detected, print:
  ```
  Prisma Accelerate Caching detected — keeping Accelerate is recommended.
  ```
- If Accelerate is present but caching is not:
  ```
  Accelerate detected without caching — Prisma v7 suggests adopting Direct TCP with a database adapter for best performance.
  ```
- Eliminate no user code; only output informational guidance.

> Produce installation commands based on the repo’s package manager.

---

## 2) Prisma Schema Changes

- In `schema.prisma`:

  - `generator client`:

    ```diff
    - provider = "prisma-client-js"
    + provider = "prisma-client"
      output   = "./generated"
    ```

  - Remove any `previewFeatures = ["driverAdapters"]` and any `engineType` attributes.

  - Update the `datasource db` block:

    - **Goal:** keep the existing `provider` value, but **remove any `url = …` entry**.

    - Example (for illustration only — do not insert comments into the user's schema):

      - Before:

        ```prisma
        datasource db {
          provider = "postgresql"
          url      = env("DATABASE_URL")
        }
        ```

      - After:

        ```prisma
        datasource db {
          provider = "postgresql"
        }
        ```

    - Rules:

      - Preserve the existing `provider` value exactly as-is (e.g. `"postgresql"`, `"mysql"`, `"sqlite"`, etc.).
      - Remove only the `url = ...` line from the `datasource db` block.
      - Preserve any other properties on the datasource (for example: `shadowDatabaseUrl`, `relationMode`, `schemas`, `extensions`, `directUrl`, etc.).
      - Do **not** add explanatory comments into the schema; comments in this prompt are hints for you, not code to emit.

- After edits, run `prisma generate`.

---

## 3) Introduce prisma.config.ts Create **prisma.config.ts** at repo root (or prisma.config.mjs), centralizing Prisma CLI config and env management:

```tsx
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
    seed: "tsx prisma/seed.ts",
  },
  datasource: {
    // Prefer DIRECT TCP via DATABASE_URL
    url: env("DATABASE_URL"),
    // Optionally support shadow DB if present:
    // shadowDatabaseUrl: env('SHADOW_DATABASE_URL'),
  },
});
```

- Remove any prisma.seed from package.json (the config above replaces it).

---

## 4) ESM & TS Baseline - Ensure **package.json**:

```json
{
  "type": "module",
  "scripts": {
    "dev": "tsx src/index.ts",
    "generate": "prisma generate",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thuansama0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
