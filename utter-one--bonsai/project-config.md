---
trigger: always_on
description: - Try to make changes in batches, not one-by-one (especially easy ones)
---

## Important Notes
- Try to make changes in batches, not one-by-one (especially easy ones)
- When changing HTTP contracts, use files in /src/http/contracts
- When changing WebSocket contracts in /src/websocket/contracts, update src/scripts/generateWebSocketSchemas.ts (integrate into one schema) and run `npm run schemas:generate` to update the JSON Schema
- Don't create example usages of newly created components
- When creating database entities, don't forget to add models
- Make all logger calls one-liners (even if very long) and do not use event field
- Do not use complex inline types. Instead create dedicated types.
- Never use require() to import type. Always use import at the beginning of files.
- Prefer types over interfaces
- Add new types of exceptions (extends Error) to /src/errors.ts
- Always create and update JSDoc comments for functions, methods and classes if necessary
- Use IoC container for classes with implicit registration by @singleton or @injectable decorator
- Always verify changes by running `npm run build` (this also regenerates WebSocket JSON Schema)
- Place private methods AFTER the public ones
- When modifying database entities (e.g. add, rename, remove fields), apply these changes to the corresponding contracts and services
- Make sure that MigrationService is synchronized with the current state of the database schema
- Do not use horizontal dividers in comments

## Database Migrations

The project uses Drizzle ORM with migrations for database schema changes.

**Key principles:**
- **Always use migrations** for schema changes (never use `db:push` in production)
- Schema changes flow: Update `/src/db/schema.ts` → Generate migration → Apply migration
- All migrations are stored in `/drizzle/` folder and version controlled

**Workflow for schema changes:**
1. Update the schema in `/src/db/schema.ts`
2. Update corresponding contracts in `/src/http/contracts/` (remove/add fields in Zod schemas)
3. Update services that reference the changed columns (column maps, queries, etc.)
4. Generate migration: `npm run db:generate`
5. Review the generated SQL in `/drizzle/XXXX_*.sql`
6. Apply migration: `npm run db:migrate`
7. Verify with `npm run build` to ensure TypeScript types align

**Important commands:**
- `npm run db:generate` - Generate a new migration based on schema changes
- `npm run db:migrate` - Apply pending migrations to the database
- `npm run db:studio` - Open Drizzle Studio to browse database

**Docker deployment:**
- Migrations run automatically on container startup before the application starts
- Migration files in `/drizzle/` are copied to the Docker image
- Connection uses `DB_CONNECTION_STRING` environment variable

## Controller Architecture

All controllers use plain Express with explicit route registration. Follow this pattern:

**Controller structure:**
```typescript
import { inject, singleton } from 'tsyringe';
import type { Request, Response, Router } from 'express';
import type { RouteConfig } from '@asteasolutions/zod-to-openapi';
import { checkPermissions } from '../../utils/permissions';
import { asyncHandler } from '../../utils/asyncHandler';

@singleton()
export class ExampleController {
  constructor(@inject(ExampleService) private readonly service: ExampleService) {}

  // Static method for OpenAPI documentation
  static getOpenAPIPaths(): RouteConfig[] {
    return [
      {
        method: 'post',
        path: '/api/examples',
        tags: ['Examples'],
        summary: 'Create example',
        description: '...',
        request: { body: { content: { 'application/json': { schema: createSchema } } } },
        responses: { 201: { description: '...', content: { 'application/json': { schema: responseSchema } } } },
      },
      // ... more routes
    ];
  }

  // Explicit route registration
  registerRoutes(router: Router): void {
    router.post('/api/examples', asyncHandler(this.createExample.bind(this)));
    router.get('/api/examples/:id', asyncHandler(this.getExample.bind(this)));
  }

  // Private handler methods
  private async createExample(req: Request, res: Response): Promise<void> {
    checkPermissions(req, [PERMISSIONS.EXAMPLE_WRITE]);
    const body = createSchema.parse(req.body);
    const result = await this.service.create(body, req.context);
    res.status(201).json(result);
  }

  private async getExample(req: Request, res: Response): Promise<void> {
    checkPermissions(req, [PERMISSIONS.EXAMPLE_READ]);
    const params = routeParamsSchema.parse(req.params);
    const result = await this.service.getById(params.id);
    res.status(200).json(result);
  }
}
```

**Key points:**
- Use `@singleton()` decorator (not `@injectable()`)
- Manual validation using `schema.parse()` in each handler
- Manual authorization using `checkPermissions(req, [PERMISSIONS.XXX])` at start of each handler
- Wrap all handlers with `asyncHandler()` to catch errors
- Private handler methods with Express signatures: `(req: Request, res: Response) => Promise<void>`
- Explicit status codes: `res.status(201).json(...)`, `res.status(204).send()`
- Static `getOpenAPIPaths()` returns OpenAPI RouteConfig array
- `registerRoutes(router: Router)` method for registering routes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utter-one/bonsai](https://github.com/utter-one/bonsai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
