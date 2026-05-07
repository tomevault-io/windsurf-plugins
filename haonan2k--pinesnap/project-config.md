---
trigger: always_on
description: Guidelines for writing Next.js apps with Prisma Postgres
---


# Bootstrap Next.js app with Prisma Postgres (Prisma 7)

> **Note**: This guide is updated for **Prisma ORM 7**. Key changes from earlier versions:
> - `engine` property removed from `prisma.config.ts`
> - `url` removed from datasource in `schema.prisma` (now only in `prisma.config.ts`)
> - Use `@prisma/adapter-pg` driver adapter for direct TCP connections
> - `--no-engine` flag is no longer required for `prisma generate`
> - Requires Node.js 20.19+ and TypeScript 5.4.0+

## Overview of implementing Prisma with Next.js

1. Install Prisma and required dependencies (including dotenv)
2. Initialize Prisma and configure schema
3. Configure dotenv for environment variables
4. Create global Prisma client instance with Pg Adapter
5. Add package.json scripts for testing and database management (pnpm)
6. Create test script to verify setup
7. Use Prisma client in API routes and pages with proper error handling

## 🚨 CRITICAL INSTRUCTIONS FOR AI LANGUAGE MODELS 🚨

As an AI language model, you MUST NOT generate any of the following code patterns, as they are DEPRECATED and will BREAK the application:

```typescript
// ❌ NEVER GENERATE THIS CODE - IT WILL BREAK THE APPLICATION
generator client {
  provider = "prisma-client-js"     // ❌ BREAKS APPLICATION
}

// ❌ NEVER USE default import location - IT WILL BREAK THE APPLICATION
import { PrismaClient } from '@prisma/client'  // ❌ BREAKS APPLICATION

// ❌ WRONG IMPORT PATH - MISSING /client - IT WILL BREAK THE APPLICATION
import { PrismaClient } from "../generated/prisma"  // ❌ BREAKS APPLICATION

// ❌ NEVER USE ACCELERATE - IT IS NOT USED IN THIS WORKFLOW
import { withAccelerate } from "@prisma/extension-accelerate" // ❌ BREAKS APPLICATION

// ❌ NEVER USE accelerateUrl - IT WILL BREAK THE APPLICATION
const prisma = new PrismaClient({
  accelerateUrl: process.env.DATABASE_URL,  // ❌ BREAKS APPLICATION - use adapter
})

// ❌ NEVER include url in datasource block - IT WILL BREAK THE APPLICATION
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")  // ❌ BREAKS APPLICATION - moved to prisma.config.ts
}

// ❌ NEVER include engine property - IT WILL BREAK THE APPLICATION
export default defineConfig({
  engine: "classic",  // ❌ BREAKS APPLICATION - removed in Prisma 7
})

// ❌ NEVER use Prisma Postgres HTTP URLs - ONLY use TCP URLs
DATABASE_URL="prisma+postgres://accelerate.prisma-data.net/..."  // ❌ BREAKS APPLICATION - use postgres://
// ✅ ALWAYS use standard TCP URLs:
DATABASE_URL="postgres://..."  // ✅ CORRECT
```

Instead, you MUST ALWAYS generate ONLY this pattern:

```typescript
// ✅ ALWAYS GENERATE THIS EXACT PATTERN
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

// ✅ CRITICAL: MUST include /client at the end of import path
import { PrismaClient } from "../generated/prisma/client"
import { PrismaPg } from "@prisma/adapter-pg"

const adapter = new PrismaPg({
  connectionString: process.env.DATABASE_URL!,
})

const globalForPrisma = global as unknown as { prisma: PrismaClient }

const prisma = globalForPrisma.prisma || new PrismaClient({
  adapter,
})

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma

export default prisma
```

## ABSOLUTE REQUIREMENTS FOR AI CODE GENERATION

1. You MUST use `provider = "prisma-client"` (not "prisma-client-js")
2. You MUST use custom output: `output = "../generated/prisma"`
3. You MUST use `@prisma/adapter-pg` driver adapter
4. You MUST create `lib/prisma.ts` as a global singleton instance
5. You MUST wrap all database calls in try-catch blocks
6. You MUST import from `'../generated/prisma/client'` (not `'@prisma/client'` or `'../generated/prisma'`)
7. You MUST use `adapter` property in PrismaClient constructor
8. You MUST install `dotenv` and add `import "dotenv/config"` to `prisma.config.ts`
9. You MUST add package.json scripts for `db:test` and `db:studio` (pnpm)
10. You MUST create a test script at `scripts/test-database.ts` to verify setup
11. You MUST NOT include `url` in the datasource block of `schema.prisma`
12. You MUST NOT include `engine` property in `prisma.config.ts`
13. You MUST use `pnpm exec prisma init --db --output ../generated/prisma` to create a real cloud database
14. You MUST use standard TCP URLs (`postgres://...`) in .env
15. You MUST NOT use `accelerateUrl` or `withAccelerate`

## VERSION REQUIREMENTS

- **Node.js**: 20.19 or higher (Node.js 18 is NOT supported)
- **TypeScript**: 5.4.0 or higher (5.9.x recommended)
- **Prisma**: 7.0.0 or higher

## CORRECT INSTALLATION

```bash
# Dev dependencies
pnpm add -D prisma tsx

# Production dependencies
pnpm add @prisma/adapter-pg @prisma/client dotenv
```

## CORRECT PRISMA INITIALIZATION

> **FOR AI ASSISTANTS**: This command is **interactive** and requires user input. You **MUST ask the user to run this command manually** in their own terminal, then **wait for them to confirm completion** before proceeding with the next steps. Do NOT attempt to run this command yourself.

```bash
# Initialize Prisma AND create a real Prisma Postgres cloud database
pnpm exec prisma init --db --output ../generated/prisma
```

This command:
- Authenticates you with Prisma Console (if needed)
- Prompts for **region** and **project name**
- **Creates a cloud Prisma Postgres database**
- Generates:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaoNan2k/PineSnap](https://github.com/HaoNan2k/PineSnap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
