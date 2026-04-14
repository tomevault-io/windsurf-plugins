---
trigger: always_on
description: EzTest is a self-hosted test management platform built with Next.js 15, React 19, TypeScript, Prisma, and PostgreSQL. It provides comprehensive test case management, test execution, defect tracking, and reporting capabilities.
---

# EzTest Cursor Rules

## Project Overview
EzTest is a self-hosted test management platform built with Next.js 15, React 19, TypeScript, Prisma, and PostgreSQL. It provides comprehensive test case management, test execution, defect tracking, and reporting capabilities.

## Architecture & Code Organization

### Frontend Structure
- **Components:** Located in `frontend/components/` organized by feature (testcase, defect, testrun, testsuite)
- **Reusable Components:** `frontend/reusable-components/` for multi-feature usage (dialogs, forms, cards, tables)
- **Base Elements:** `frontend/reusable-elements/` or `components/ui/` for primitive UI components (buttons, inputs, badges)
- **Styling:** Tailwind CSS v4 + Radix UI + Glass Morphism design pattern

### Backend Structure
- **API Routes:** `app/api/**/route.ts` - Next.js App Router REST endpoints
- **Controllers:** `backend/controllers/` - Request validation and response formatting
- **Services:** `backend/services/` - Business logic and data operations
- **Validators:** `backend/validators/` - Zod schemas for input validation
- **Database:** Prisma ORM with PostgreSQL 16

### Request Flow
```
Client Request → API Route → Controller → Service → Prisma → Database
```

## Development Guidelines

### File Organization & Naming
- **Components:** PascalCase (e.g., `TestCaseDialog.tsx`)
- **Utilities/Hooks:** camelCase (e.g., `useTestCase.ts`, `formatDate.ts`)
- **API Routes:** RESTful paths matching resource names (e.g., `/api/testcases/[id]/route.ts`)
- **Services/Controllers:** One file per resource (e.g., `backend/services/testcase/index.ts`)

### Import Patterns
```typescript
// Use @ alias for absolute imports
import { testCaseService } from '@/backend/services/testcase';
import { TestCaseDialog } from '@/frontend/components/testcase';
import { Button } from '@/components/ui/button';

// Group imports: React → External → Internal
import React, { useState } from 'react';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
```

### Server vs Client Components
- **Default:** Server Components for data fetching and security
- **"use client"** for: interactive forms, dialogs, state management, event handlers
- Always keep auth logic server-side (lib/auth, lib/rbac)

## Authentication & Authorization

### Auth Implementation
- **Method:** NextAuth.js with JWT in httpOnly cookies
- **Password Hashing:** bcryptjs v3.0.2
- **Session:** Stateless JWT containing user permissions
- **RBAC:** Role-based access control with fine-grained permissions

### Authorization Pattern
```typescript
// In API routes (always server-side)
import { getSessionUser } from '@/lib/auth/getSessionUser';
import { hasPermission } from '@/lib/rbac';

const user = await getSessionUser();
if (!hasPermission(user, 'manage_projects')) {
  return NextResponse.json({ error: 'Unauthorized' }, { status: 403 });
}
```

### API Key Auth
For programmatic access:
```typescript
import { apiKeyAuth } from '@/lib/auth/apiKeyAuth';
const user = await apiKeyAuth(request.headers.get('x-api-key'));
```

## Database

### Schema Location
- `prisma/schema.prisma` - All data models
- `prisma/migrations/` - Historical migrations
- Key Models: User, Project, TestCase, TestStep, TestSuite, Module, TestRun, Defect, Attachment

### Key Pattern: Dynamic Enums
Enums have been replaced with `DropdownOption` table for dynamic management. Don't add static enums to schema.

### Prisma Usage
```typescript
import { prisma } from '@/lib/prisma';

// Always use Prisma for database operations
const testCase = await prisma.testCase.findUnique({
  where: { id },
  include: { steps: true, attachments: true }
});
```

### Migrations
```bash
npx prisma migrate dev --name description  # Create migration
npx prisma migrate deploy                  # Apply migrations
npx prisma studio                          # Visual explorer
```

## API Response Format
```typescript
// Successful response
{
  status: 'success',
  data: { /* resource data */ },
  message: 'Optional success message'
}

// Error response
{
  status: 'error',
  error: 'Error description',
  message: 'User-friendly message'
}
```

## Validation & Error Handling

### Input Validation
Always use Zod for runtime validation:
```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1, 'Name is required'),
  description: z.string().optional(),
  steps: z.array(z.object({ title: z.string() }))
});

const data = schema.parse(req.body);
```

### HTTP Status Codes
- **200/201:** Success
- **400:** Bad request (validation error)
- **401:** Unauthenticated
- **403:** Unauthorized (permission denied)
- **404:** Not found
- **500:** Server error

## Feature: File Attachments

### Storage Options
- AWS S3 (preferred for production)
- Local directory (development)

### Multipart Upload
Large files use chunked uploads via AWS S3 multipart API. Endpoints:
- `POST /api/attachments/upload` - Initiate
- `POST /api/attachments/upload/part` - Upload chunk
- `POST /api/attachments/upload/complete` - Finalize

### Usage
```typescript
// Test case attachments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/houseoffoss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
