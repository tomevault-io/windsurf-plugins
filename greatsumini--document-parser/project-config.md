---
trigger: always_on
description: This project is a cloud-native SaaS web application that enables users to upload various data files (Excel, CSV, PDF, TXT) and automatically generate hospital-compliant Markdown documents using LLM (OpenAI GPT-4o). The stack comprises Next.js 15 (App Router, server components), TypeScript, tRPC, Supabase (PostgreSQL & Storage), OpenAI SDK, Vercel AI SDK, Shadcn UI, Tailwind CSS, and modern DevOps (Vercel, GitHub Actions, Sentry).
---

# Project Code Guideline

---

## 1. Project Overview

This project is a cloud-native SaaS web application that enables users to upload various data files (Excel, CSV, PDF, TXT) and automatically generate hospital-compliant Markdown documents using LLM (OpenAI GPT-4o). The stack comprises Next.js 15 (App Router, server components), TypeScript, tRPC, Supabase (PostgreSQL & Storage), OpenAI SDK, Vercel AI SDK, Shadcn UI, Tailwind CSS, and modern DevOps (Vercel, GitHub Actions, Sentry).  
**Key architectural decisions:**
- Monorepo structure with domain-driven organization.
- Serverless-first deployment (Vercel Edge/Serverless, Supabase).
- Strong boundaries between presentation, business logic, and infrastructure.
- Type safety and input validation across the stack (TypeScript, Zod).
- Real-time, streaming, and batch data flows.

---

## 2. Core Principles

1. **Type Safety First:** All code MUST use TypeScript with strict type checks enabled.
2. **Single Responsibility:** Each module/component MUST have only one clear responsibility.
3. **Explicit Error Handling:** All external calls and user input MUST be validated and errors handled gracefully.
4. **Domain-Centric Organization:** Code MUST be organized by business domain, not by technical layer.
5. **Security by Default:** Sensitive operations and data access MUST follow least privilege and encryption requirements.

---

## 3. Language-Specific Guidelines

### 3.1 TypeScript & Next.js

#### File Organization and Directory Structure

- **MUST:** Follow the monorepo and domain-driven folder structure as specified:

    ```
    /apps/web/
      app/                  // Next.js app router structure
      components/           // Reusable UI components
      features/{domain}/    // Domain features (e.g., report, template, auth)
      lib/                  // Utilities and API clients
      styles/               // Tailwind and global styles
    /packages/
      api/                  // tRPC routers and procedures
      db/                   // Prisma schema and DB utilities
      llm/                  // LLM prompt helpers and adapters
      shared/types/         // Shared TypeScript types
      shared/utils/         // Shared utility functions
      shared/constants/     // Shared constants
    /infra/                 // Deployment, scripts, config
    ```

- **MUST:** Place each feature in its own folder under `/features/{domain}` with clear separation of UI, hooks, and logic.

```typescript
// MUST: Example feature folder structure
/features/report/
  ReportEditor.tsx
  useReportData.ts
  reportUtils.ts
```

- **MUST NOT:** Place unrelated components, utilities, or logic in a single file or folder.

#### Import/Dependency Management

- **MUST:** Use absolute imports from the project root, not relative paths traversing multiple directories.

```typescript
// MUST: Use absolute imports for clarity and maintainability
import { parseExcel } from 'features/report/reportUtils'
```

- **MUST:** Group external imports before internal imports, and order alphabetically within each group.

- **MUST:** Keep dependencies minimal and only import what is required for the module’s responsibility.

- **MUST NOT:** Use wildcard imports (`import * as ...`) except for TypeScript enums or namespaces.

#### Error Handling Patterns

- **MUST:** Validate all user inputs and API payloads using Zod schemas, both client- and server-side.

```typescript
// MUST: Zod validation for incoming API data
import { z } from 'zod'

const UploadSchema = z.object({
  file: z.instanceof(File),
  templateId: z.string().uuid(),
})

export const uploadHandler = (data: unknown) => {
  const parsed = UploadSchema.safeParse(data)
  if (!parsed.success) {
    throw new Error('Invalid input data')
  }
  // Proceed with validated data
}
```

- **MUST:** Handle all async errors using try/catch and surface user-friendly messages.

```typescript
// MUST: Graceful async error handling
try {
  const result = await api.createDocument(payload)
} catch (error) {
  logger.error(error)
  showToast('Failed to create document. Please try again.')
}
```

- **MUST NOT:** Swallow errors or use empty catch blocks.

---

### 3.2 tRPC & API Layer

- **MUST:** Separate routers by domain (`reportRouter`, `templateRouter`, `authRouter`).
- **MUST:** Define input/output types using Zod and TypeScript generics.
- **MUST:** Use context for authentication and authorization checks on each procedure.
- **MUST NOT:** Mix unrelated procedures in the same router or expose raw database models directly.

---

### 3.3 Prisma & Supabase Integration

- **MUST:** Use Prisma Client for all database access.
- **MUST:** Use transactions for multi-step DB operations (e.g., document + version record).
- **MUST:** Never expose raw SQL queries in business logic.
- **MUST:** Use Row Level Security (RLS) for all data access.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greatSumini/document-parser](https://github.com/greatSumini/document-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
