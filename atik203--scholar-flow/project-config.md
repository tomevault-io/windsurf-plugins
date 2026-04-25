---
trigger: always_on
description: Backend architecture, database operations, and authentication
---


# Backend Rules

## ✅ ESTABLISHED SYSTEMS (Phase 1 – Week 5.5)

1. **Authentication**: OAuth (Google/GitHub) + JWT system with upsert handling ✅
2. **Paper Management**: Upload, S3 storage, metadata extraction, AI summarization cache ✅
3. **User Profiles & Workspaces**: CRUD, permissions, invitations, activity logging ✅
4. **AI Insights**: Gemini-first + OpenAI fallback service powering summaries/chat ✅
5. **Production Readiness**: Security, monitoring, health checks, rate limiting ✅

## Architecture & Structure

1. **Structure**: Feature modules under `src/app/modules` (controller, service, routes, validation schemas).
2. **Routes**: Register in `src/app/routes` only. No ad-hoc route registration elsewhere.
3. **Controllers**: Thin. Business logic belongs in services.
4. **Services**: Use Prisma client from shared singleton. Handle transactions explicitly when needed.
5. **Validation**: Zod for request bodies, params, queries. Reject early on failure.

## Database Operations - $queryRaw Optimization

**CRITICAL RULE**: All database operations MUST use `$queryRaw` or `$executeRaw` instead of Prisma Client methods for optimal performance.

### ✅ REQUIRED: Use $queryRaw for:

- All SELECT queries (findMany, findUnique, findFirst)
- All INSERT operations (create)
- All UPDATE operations (update, upsert)
- All DELETE operations (delete)
- All aggregation queries (count, groupBy)
- All complex joins and relationships

### 🔒 EXCEPTIONS: Keep Prisma Client ONLY for:

- **OAuth operations** that require complex constraint handling:
  - `createAccount()` - Provider constraint management
  - `createOrUpdateUser()` - Email constraint management
  - `createOrUpdateUserWithOAuth()` - Email verification handling

### Implementation Standards

```typescript
// ✅ CORRECT: Use $queryRaw with proper typing
const users = await prisma.$queryRaw<any[]>`
  SELECT id, email, name, "createdAt"
  FROM "User"
  WHERE "isDeleted" = false
  ORDER BY "createdAt" DESC
  LIMIT ${limit} OFFSET ${skip}
`;

// ❌ WRONG: Using Prisma Client
const users = await prisma.user.findMany({
  where: { isDeleted: false },
  orderBy: { createdAt: "desc" },
  skip,
  take: limit,
});
```

### Security & Performance

- **SQL Injection Prevention**: Use template literals: `WHERE email = ${email}`
- **Type Safety**: Use `prisma.$queryRaw<Type[]>` for proper typing
- **Field Selection**: Select only required fields, avoid `SELECT *`
- **Pagination**: Use `LIMIT ${limit} OFFSET ${skip}` for efficient pagination

## Authentication & Security

1. **OAuth Account Management**: ALWAYS use standard Prisma upsert for OAuth accounts.
2. **Raw Query Warning**: Do NOT change OAuth account creation to raw queries - causes unique constraint errors (P2002).
3. **Error Handling**: All auth methods must use try/catch with ApiError for consistent error responses.
4. **JWT**: Use JWT access + refresh. Use `auth` middleware for protected endpoints.
5. **CORS**: Use `FRONTEND_URL` env; default `http://localhost:3000` in dev.

## 🚀 Production-Ready Patterns (September 17, 2025)

### Security & Performance Standards

1. **Debug Logging Control**: Auth middleware conditionally logs only in development
   ```typescript
   // ✅ CORRECT: Conditional debug logging
   if (process.env.NODE_ENV !== "production") {
     console.log("Auth debug info:", { userId, role });
   }
   ```

2. **Type Safety**: Replace all `any` types with proper interfaces
   ```typescript
   // ✅ CORRECT: Use AuthenticatedRequest interface
   export interface AuthenticatedRequest extends Request {
     user: { id: string; email: string; role: string };
   }
   ```

3. **Rate Limiting**: Apply feature-specific rate limiters to endpoints
   ```typescript
   // ✅ CORRECT: Feature-specific rate limiters
   import { paperUploadLimiter, paperListLimiter } from '../middleware/rateLimiter';
   router.post('/upload', paperUploadLimiter, uploadPaper);
   router.get('/', paperListLimiter, listPapers);
   ```

4. **Database Performance**: Add composite indexes for hot query paths
   ```prisma
   // ✅ CORRECT: Composite indexes for performance
   model Paper {
     @@index([uploaderId, workspaceId], name: "paper_uploader_workspace_idx")
     @@index([processingStatus, createdAt], name: "paper_status_created_idx")
   }
   ```

### Error Handling Standards

1. **Feature-Specific Error Classes**: Create error classes extending `ApiError`
   ```typescript
   // ✅ CORRECT: Feature-specific error handling
   export class PaperError extends ApiError {
     constructor(message: string, statusCode: number = 400, errorCode?: string) {
       super(statusCode, message);
       this.errorCode = errorCode;
     }
   }
   ```

2. **Standardized Error Format**: All errors follow consistent structure
   ```typescript
   // ✅ CORRECT: Standardized error response
   {
     success: false,
     message: string,
     statusCode: number,
     errorCode?: string,
     details?: object,
     timestamp: string
   }
   ```

### Performance Monitoring

1. **Performance Monitoring Middleware**: Track response times and add headers
   ```typescript
   // ✅ CORRECT: Performance monitoring
   import { performanceMonitor } from '../middleware/performanceMonitor';
   app.use(performanceMonitor);
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Atik203) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
