---
trigger: always_on
description: - **Framework**: Remix.run (React Router v7) with React - imports you may expect to come from '@remix-run/react|node' actually come from 'react-router'
---

# Project Technologies and Patterns

## Basic Technologies

- **Framework**: Remix.run (React Router v7) with React - imports you may expect to come from '@remix-run/react|node' actually come from 'react-router'
- **Database**: PostgreSQL with Kysely for type-safe queries and Kanel for database schema codegen
- **API Layer**: ORPC (similar to tRPC) for type-safe server-client communication and server-side RPC calls
- **UI Components**: Tailwind CSS with shadcn/ui components
- **Data Fetching**: React Query for client-side data management
- **Testing**: Vitest with isolated wasm-backed databases per test

## UI Components

All shadcn/ui components can be available in `app/components/ui/`. Just run `bunx shadcn-ui@latest add <x>` to add a new component.

Do not directly add components to `app/components/ui`. That is only for components from shadcn/ui. Our components go elsewhere in the components directory.

I use @ as the root indicator, so import from `@/components/ui/card` instead of `~/components/ui/card`.

## Adding a Route

1. Create a new file in `app/routes/` with the route name (e.g., `my-route.tsx`)
2. Add an entry to `app/routes.ts`
3. Structure your route with:
   ```typescript
   import type { Route } from "./+types/my-route"; // my-route is the name of the route file

   export async function loader({ params }: Route.LoaderArgs) {
     // Server-side data loading
     return {
       // Your data
     };
   }

   export default function MyRoute({ loaderData }: Route.ComponentProps) {
     // Your component
   }
   ```

## Database Migrations and Codegen

1. Add migrations in `app/database/migrations/current.sql`
2. Run codegen using `bun codegen`
3. Database types will be available in `app/database/types/`

### Important Database Schema Notes

- **Table References**: When using Kysely queries, do NOT include schema names (e.g., `my_schema.`)
- ✅ Correct: `.selectFrom("form_submission")`
- ❌ Incorrect: `.selectFrom("uaw.form_submission")`
- The database configuration automatically handles schema resolution
- Schema names are only needed in migration SQL files, not in TypeScript code

## Data Loading Patterns

**Important**: The `withPrefetch` function provides three parameters:
- `queryClient` - React Query client for prefetching
- `orpc` - Direct ORPC caller for server-side operations. Call procedures directly as a function
- `orpcQuery` - ORPC React Query utils for generating query options. Used for prefetching queries on the client.

### When to use each approach:

1. **Loader Data (Server-Side)**
   - Use for static data that won't change during the lifetime of the page
   - Initial page load data that's critical for rendering

   Example:
   ```typescript
   // In app/orpc/user.server.ts
   export const userRouter = base.router({
     getUser: base
       .input(z.object({ userId: z.string() }))
       .handler(async ({ input, context }) => {
         return await context.db
           .selectFrom("users")
           .where("id", "=", input.userId)
           .executeTakeFirst();
       }),
   });

   // In your route file
   export async function loader({ params }: Route.LoaderArgs) {
     const user = await orpcCaller.user.getUser({ userId: params.userId });
     return { user };
   }

   export default function UserRoute({ loaderData }: Route.ComponentProps) {
     const { user } = loaderData;
     return <div>Hello {user.name}!</div>;
   }
   ```

2. **React Query with Prefetch**
   - Dynamic data needed on initial page load
   - Data that might change while the page is open
   - Use `prefetchQuery` in the loader for optimal loading
   - Data requirements that are most cleanly defined in nested components

   Example:
   ```typescript
   // In app/orpc/posts.server.ts
   export const postsRouter = base.router({
     getLatestPosts: base.handler(async ({ context }) => {
       return await context.db
         .selectFrom("posts")
         .orderBy("created_at", "desc")
         .limit(10)
         .execute();
     }),
   });

   // In your route file
   import { withPrefetch } from "@/lib/orpcCaller.server";
   export async function loader({ context }: Route.LoaderArgs) {
     return withPrefetch<{ someData: string }>(async (queryClient, orpc, orpcQuery) => {
       // This prefetches the query for client-side use
       await queryClient.prefetchQuery(
         orpcQuery.posts.getLatestPosts.queryOptions({
           input: {} // Pass any required input parameters here
         })
       );
       
       // Can return other data if desired
       return { someData: "example" };
     });
   }

   export default function PostsRoute({ loaderData }: Route.ComponentProps) {
     const { result: { someData } } = loaderData;
     
     // Data is already prefetched and available immediately
     const { data: posts } = useQuery(orpcFetchQuery.posts.getLatestPosts.queryOptions());

     return (
       <div>
         {posts.map(post => <PostCard key={post.id} post={post} />)}
       </div>
     );
   }
   ```

3. **Client-Side React Query**
   - Non-critical data that can load after initial render
   - User-triggered data fetches
   - Polling or real-time updates

   Example:
   ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ben-pr-p) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
