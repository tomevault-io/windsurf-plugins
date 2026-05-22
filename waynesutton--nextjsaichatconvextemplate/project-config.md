---
trigger: always_on
description: Expereinced Convex and Clerk develope
---

# Convex guidelines
---
alwaysApply: true
---


## Function guidelines
- follow Convex schemas best practices - https://docs.convex.dev/database/schemas
- check the convex schema for updates and errors
- understand Convex - https://docs.convex.dev/understanding/
- understand environment-variables https://docs.convex.dev/production/environment-variables
- understand and follow Convex best-practices https://docs.convex.dev/understanding/best-practices/
- Follow best practices https://docs.convex.dev/understanding/best-practices/typescript
- Always use query-functions https://docs.convex.dev/functions/query-functions
-  Always usehttps://docs.convex.dev/functions/mutation-functions
- expert https://docs.convex.dev/functions/mutation-functions
-  Always use https://docs.convex.dev/functions/actions
- knows https://docs.convex.dev/functions/validation
- knows https://docs.convex.dev/functions
- Expert in Clerk https://docs.clerk.com/
- you are an expert in understanding how  Uploading and Storing Files with convex  https://docs.convex.dev/file-storage/upload-files
- you are an expert in convex auth - https://docs.convex.dev/auth/convex-auth
- you are an expert in setting up convex auth https://labs.convex.dev/auth/setup
- you an an expert in convex vector search https://docs.convex.dev/search/vector-search
getting-started
- do not use emoji in the readme or app
- do not over engineer the code but make it typesafe 
- do not do more than what the user ask for unless it related to fixing, adding, or updating the code to what the user is asking for
  
### New function syntax
- ALWAYS use the new function syntax for Convex functions. For example:
      ```typescript
      import { query } from "./_generated/server";
      import { v } from "convex/values";
      export const f = query({
          args: {},
          returns: v.null(),
          handler: async (ctx, args) => {
          // Function body
          },
      });
      ```

### Http endpoint syntax
- HTTP endpoints are defined in `convex/http.ts` and require an `httpAction` decorator. For example:
      ```typescript
      import { httpRouter } from "convex/server";
      import { httpAction } from "./_generated/server";
      const http = httpRouter();
      http.route({
          path: "/echo",
          method: "POST",
          handler: httpAction(async (ctx, req) => {
          const body = await req.bytes();
          return new Response(body, { status: 200 });
          }),
      });
      ```

### Function registration
- Use `internalQuery`, `internalMutation`, and `internalAction` to register internal functions. These functions are private and aren't part of an app's API. They can only be called by other Convex functions.
- Use `query`, `mutation`, and `action` to register public functions. These functions are part of the public API and are exposed to the public Internet. Do NOT use `query`, `mutation`, or `action` to register sensitive internal functions that should be kept private.
- You CANNOT register a function through the `api` or `internal` objects.
- ALWAYS include argument and return validators for all Convex functions. If a function doesn't return anything, include `returns: v.null()` as its output validator.
- If the JavaScript implementation of a Convex function doesn't have a return value, it implicitly returns `null`.

### Function calling
- Use `ctx.runQuery` to call a query from a query, mutation, or action.
- Use `ctx.runMutation` to call a mutation from a mutation or action.
- Use `ctx.runAction` to call an action from an action.
- ONLY call an action from another action if you need to cross runtimes (e.g. from V8 to Node). Otherwise, pull out the shared code into a helper async function and call that directly instead.
- Try to use as few calls from actions to queries and mutations as possible. Queries and mutations are transactions, so splitting logic up into multiple calls introduces the risk of race conditions.
- All of these calls take in a `FunctionReference`. Do NOT try to pass the callee function directly into one of these calls.
- When using `ctx.runQuery`, `ctx.runMutation`, or `ctx.runAction` to call a function in the same file, specify a type annotation on the return value to work around TypeScript circularity limitations. For example,
                            ```
                            export const f = query({
                              args: { name: v.string() },
                              returns: v.string(),
                              handler: async (ctx, args) => {
                                return "Hello " + args.name;
                              },
                            });

                            export const g = query({
                              args: {},
                              returns: v.null(),
                              handler: async (ctx, args) => {
                                const result: string = await ctx.runQuery(api.example.f, { name: "Bob" });
                                return null;
                              },
                            });
                            ```

### Function references
- Function references are pointers to registered Convex functions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/nextjsaichatconvextemplate](https://github.com/waynesutton/nextjsaichatconvextemplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
