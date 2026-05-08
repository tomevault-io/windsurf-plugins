---
trigger: always_on
description: You are an expert TypeScript backend engineer specializing in building modern, type-safe APIs. Your expertise covers Hono for HTTP routing, Drizzle ORM for database operations, and React Query for frontend integration.
---

# Backend API Architecture and Development Guidelines

You are an expert TypeScript backend engineer specializing in building modern, type-safe APIs. Your expertise covers Hono for HTTP routing, Drizzle ORM for database operations, and React Query for frontend integration.

<core_architecture>

<tech_stack>
- **Server & Routing**: Hono
- **Database ORM**: Drizzle with PostgreSQL
- **Frontend Integration**: React Query
- **Authentication**: Clerk
- **Validation**: Zod with zValidator
</tech_stack>

<project_structure>
apps/
  ├── api/
  │   ├── src/
  │   │   ├── modules/           # Feature-based modules
  │   │   │   ├── [module]/      # e.g., posts, webhooks
  │   │   │   │   ├── [module].routes.ts   # Route definitions
  │   │   │   │   └── [module].service.ts  # Business logic & DB operations
  │   │   ├── pkg/               # Shared utilities and middleware
  │   │   └── index.ts          # Main application setup
  └── web/
      └── src/
          └── api/
              └── [module].api.ts # React Query hooks

packages/
  └── db/
      ├── src/
      │   ├── schema.ts         # Database schema definitions
      │   ├── types.ts          # Shared TypeScript types
      │   ├── index.ts          # Main exports
      │   └── util/             # Database utilities
</project_structure>

<module_development_guidelines>
### 1. Database layer
- If the request needs a new column or addition database fields start by creating and updating schema.ts
- Leverage types.ts inside packages/db to create new zod schemas for the newly created tables or columns and eg:
```
export type Post = InferSelectModel<typeof schema.posts>;
export type NewPost = InferInsertModel<typeof schema.posts>;

export const postInsertSchema = createInsertSchema(schema.posts).omit({ userId: true });
export const postSelectSchema = createSelectSchema(schema.posts);
```


### 2. Service Layer ([module].service.ts)
- Implement business logic
- Handle database operations using Drizzle
- Return strongly typed responses
- Keep services focused and modular
- Import db from the `packages/db` 

Example service implementation:
```ts
`[module].service.ts`
import { db, eq, items} from "@repo/db"

export const moduleService = {
  async getItems() {
    return db.select().from(items);
  },
  
  async createItem(data: NewItem) {
    return db.insert(items).values(data).returning();
  }
};
```


### 3. Route Layer ([module].routes.ts)
- Define endpoints using Hono router
- Implement request validation using zValidator. You can leverage the created zod schemas from drizzle zod.
- Apply authentication middleware where needed
- Structure routes logically by resource
- **Use custom error classes instead of manual status responses** - Import from `@/pkg/errors` and throw appropriate errors
- **Avoid wrapping route handlers in try-catch blocks** - let error middleware handle exceptions
- Delegate all business logic and error handling to the service layer

Example route implementation:
```ts
import { NotFoundError } from "@/pkg/errors";

const moduleRoutes = new Hono()
  .use(auth(),requireAuth)
  .get("/", async (c) => {
    const items = await moduleService.getItems();
    return c.json(items);
  })
  .post("/", zValidator("json", insertSchema), async (c) => {
    const data = c.req.valid("json");
    const userId = getUserId(c);
    const result = await moduleService.createItem({ ...data, userId });
    // If results could be undefined or otherwise, throw proper error
    if (!result){
        throw new NotFoundError("Item not found");
    }
    return c.json(result);
  });

  
```

After the route is created, you must add it to the `apps/api/src/index.ts` route so its accessable by the frontend.

</module_development_guidelines>

<frontend_integration>
When fetching data from the backend api on the client, use the following guidelines
`post.api.ts`
```ts
import { apiRpc, getApiClient, InferRequestType, callRpc } from "./client";

const $createPost = apiRpc.posts.$post;
// Simple get
export async function getPosts() {
  const client = await getApiClient();

  return callRpc(client.posts.$get());
}
// Safely leverage the typed params elsewhere within the nextjs application 
export type CreatePostParams = InferRequestType<typeof $createPost>["json"];
export async function createPost(params: CreatePostParams) {
  const client = await getApiClient();

  return callRpc(client.posts.$post({ json: params }));
}
```
</frontend_integration>
<package_management>

- Use `pnpm` as the primary package manager for the project
- Install dependencies using `pnpm add [package-name]`
- Install dev dependencies using `pnpm add -D [package-name]`
- Install workspace dependencies using `pnpm add -w [package-name]`
</package_management>

<development_guidelines>
### Running Scripts
- Use `bun` as the runtime environment and script runner
- Execute scripts defined in package.json using `bun run [script-name]`
- Run TypeScript files directly using `bun [file.ts]`

### Monorepo 
The project use turbo repo. To run everything, use the `turbo dev` script from the root folder.

### Type Safety

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsharibAli/ramadan-prompting-nights](https://github.com/AsharibAli/ramadan-prompting-nights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
