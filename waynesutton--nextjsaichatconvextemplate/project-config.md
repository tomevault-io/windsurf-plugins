---
trigger: always_on
description: export const convexGuidelines = `# Convex guidelines
---

export const convexGuidelines = `# Convex guidelines
## Function guidelines
### Http endpoint syntax
- HTTP endpoints are defined in \`convex/http.ts\` and require an \`httpAction\` decorator. For example:
      \`\`\`typescript
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
      \`\`\`
- HTTP endpoints are always registered at the exact path you specify in the \`path\` field. For example, if you specify \`/api/someRoute\`, the endpoint will be registered at \`/api/someRoute\`.

### Validators
- Below is an example of an array validator:
                            \`\`\`typescript
                            import { mutation } from "./_generated/server";
                            import { v } from "convex/values";

                            export default mutation({
                            args: {
                                simpleArray: v.array(v.union(v.string(), v.number())),
                            },
                            handler: async (ctx, args) => {
                                //...
                            },
                            });
                            \`\`\`
- Below is an example of a schema with validators that codify a discriminated union type:
                            \`\`\`typescript
                            import { defineSchema, defineTable } from "convex/server";
                            import { v } from "convex/values";

                            export default defineSchema({
                                results: defineTable(
                                    v.union(
                                        v.object({
                                            kind: v.literal("error"),
                                            errorMessage: v.string(),
                                        }),
                                        v.object({
                                            kind: v.literal("success"),
                                            value: v.number(),
                                        }),
                                    ),
                                )
                            });
                            \`\`\`
- Always use the \`v.null()\` validator when returning a null value. Below is an example query that returns a null value:
                                  \`\`\`typescript
                                  import { query } from "./_generated/server";
                                  import { v } from "convex/values";

                                  export const exampleQuery = query({
                                    args: {},
                                    returns: v.null(),
                                    handler: async (ctx, args) => {
                                        console.log("This query returns a null value");
                                        return null;
                                    },
                                  });
                                  \`\`\`
The full list of Convex validators is v.id("tablename"), v.null(), v.number(), v.int64(), v.boolean(), v.string(), v.bytes(), v.array(), v.object(), and v.record().

v.object(), v.array, v.boolean, v.number, v.string, v.id, and v.null are the most common validators you'll need.

### Function registration
- Use \`internalQuery\`, \`internalMutation\`, and \`internalAction\` to register internal functions. These functions are private and aren't part of an app's API. They can only be called by other Convex functions. These functions are always imported from \`./_generated/server\`.
- Use \`query\`, \`mutation\`, and \`action\` to register public functions. These functions are part of the public API and are exposed to the public Internet. Do NOT use \`query\`, \`mutation\`, or \`action\` to register sensitive internal functions that should be kept private.
- You CANNOT register a function through the \`api\` or \`internal\` objects.
- ALWAYS include argument and return validators for all Convex functions. This includes all of \`query\`, \`internalQuery\`, \`mutation\`, \`internalMutation\`, \`action\`, and \`internalAction\`. If a function doesn't return anything, include \`returns: v.null()\` as its output validator.
- If the JavaScript implementation of a Convex function doesn't have a return value, it implicitly returns \`null\`.

### Api design
- Convex uses file-based routing, so thoughtfully organize files with public query, mutation, or action functions within the \`convex/\` directory.
- Use \`query\`, \`mutation\`, and \`action\` to define public functions.
- Use \`internalQuery\`, \`internalMutation\`, and \`internalAction\` to define private, internal functions.

### Pagination
- Paginated queries are queries that return a list of results in incremental pages.
- You can define pagination using the following syntax:

                            \`\`\`ts
                            import { v } from "convex/values";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/nextjsaichatconvextemplate](https://github.com/waynesutton/nextjsaichatconvextemplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
