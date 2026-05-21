---
trigger: always_on
description: When working in the controller app
---


# Controller App

## Overview
- This is a Bun/NodeJS app.
- We use a standard flow: `x.routes.ts` -> `x.service.ts` -> `x.repository.ts`
- For external API calls, use `x.api-client.ts`

### Specific Questions
- When asked to create a new API route, we mean creating an api definition, routes implementation, service method and repository method with a database query.

## Dependency Injection
- Uses `npm:typed-inject`
- Injection context defined in [injector.ts](mdc:apps/controller/src/injector.ts)
- Key principles:
  - Use interfaces for injectables
  - Classes inject other classes
  - All injectable classes (with `static inject`) must be added to `injector.ts`
- When creating a new file that is injectable, ALWAYS make sure to add it to injector.ts

Example injectable class with authentication:
```ts
import { tokens } from "typed-inject";
import { createRoute } from "@hono/zod-openapi";
import type { IAuthenticationMiddleware } from "@/authentication/authentication.middleware.ts";

export interface IMyService {
  createThing(thing: Thing): Promise<ThingEntity>;
}

export class ExampleRoutes {
  static inject = tokens("myService", "authenticationMiddleware");

  #routes;

  constructor(
    myService: IMyService,
    authenticationMiddleware: IAuthenticationMiddleware,
  ) {
    this.#routes = new OpenAPIHono()
      .openapi(
        createRoute({
          ...myThingCreateApi,
          middleware: [authenticationMiddleware.requireLogin()] as const,
        }),
        async (c) => {
          const { organizationId } = c.req.valid("param");
          const clerkUserId = c.get("clerkUserId");
          await authenticationMiddleware.requireOrganizationAccess(clerkUserId, organizationId);

          const thing = c.req.valid("json");
          const result = await myService.createThing(thing);
          return c.json(result, 201);
        },
      );
  }

  get routes() {
    return this.#routes;
  }
}
```

### Identifiers (ids) and codes
- We use two types of identifiers:
  1. External IDs (codes): Stes used in APIs and services (e.g., "ORG_123", "PERS_456")
  2. Internal IDs: Numeric values used in the database and repositories
- Naming conventions:
  - In API definitions and service layer: Use `organizationId` for the external code
  - In repository layer: 
    - Use `organizationCode` when referring to the external code
    - Use `organizationId` when referring to the internal numeric ID
  - In database schema:
    - `code` column for external IDs (string)
    - `id` column for internal IDs (number)
- When joining tables in repositories:
  - Service methods receive external codes (`organizationId`)
  - Repository methods accept external codes (`organizationCode`)
  - Repository joins with organization table to get internal IDs
  - Example:
    ```ts
    // Service
    async update(params: { organizationId: string, slug: string }) {
      // organizationId here is the external code
      await repository.update({ organizationCode: params.organizationId, slug });
    }

    // Repository
    async update(params: { organizationCode: string, slug: string }) {
      // Join with organization to get internal ID
      .innerJoin(
        schema.organization,
        and(
          eq(schema.organization.id, schema.thing.organizationId),
          eq(schema.organization.code, params.organizationCode),
        ),
      )
    }
    ```

## Database & ORM
### Drizzle Usage
- Drizzle is used as ORM/Query Builder
- One Typescript method in a repository class should generally have ONE Postgres query. Having multiple ones is only allowed for very complex write operations.
- If a query is complicated, first write the query in pure Postgres. Then rewrite with Drizzle's query builder.
- Use CTEs with `with` and `$with` as needed
- Schema defined in [schema.ts](mdc:apps/controller/src/postgres/schema.ts)
- In repositories BE VERY MINDFUL about ids and codes. Usually we are only handed a string (code) from the API, the query is responsible for getting the assoicated id.
- When doing multiple mutations in a single repository method, ALWAYS make sure they're in a transaction.

## Services & Repositories
- Services and repositories generally have similar methods.
- The types these methods take however, are per repository and service.
  - Repositories deal with more private data. The service layer converts those to public data.
- When at all possible DO NOT export types from a repository or service. The only reason to export is when the type might be needed in tests.
- PREFER to recreate types instead of importing them.

### Repository Pattern
Example repository:
```ts
import { tokens } from "typed-inject";
import type { PostgresManager } from "@/postgres/postgres.ts";

export interface ISomeRepository {
  // ...
}

export class SomeRepository implements ISomeRepository {
  static inject = tokens("postgres");

  #db: PostgresManager["db"];

  constructor(postgres: PostgresManager) {
    this.#db = postgres.db;
  }

  async createThing(someCode: string): Promise<Thing> {
    const org = this.#db.$with("org").as(
      this.#db
        .select({
          id: schema.organization.id,
          code: schema.organization.code,
          name: schema.organization.name,
        })
        .from(schema.organization)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rejot-dev/rejot](https://github.com/rejot-dev/rejot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
