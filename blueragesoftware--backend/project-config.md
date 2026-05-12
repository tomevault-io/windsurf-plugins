---
trigger: always_on
description: We are creating an execution environment for AI Agents. We believe that instead of complex programming, users simply describe what they want accomplished, and our platform handles the rest. A declartive approach. Anyone can create AI agents by connecting their existing applications via tools and assembling a workflow. These agents run automatically based on schedules or triggers, and can be easily shared.
---

# AI Agents instructions

## Project description

We are creating an execution environment for AI Agents. We believe that instead of complex programming, users simply describe what they want accomplished, and our platform handles the rest. A declartive approach. Anyone can create AI agents by connecting their existing applications via tools and assembling a workflow. These agents run automatically based on schedules or triggers, and can be easily shared.
Our iOS app makes it seamless to share content with your agents and trigger workflows on-the-go. We serve both consumers looking to automate personal tasks and businesses needing regular automation. It's a platform designed for rapid experimentation—launch small automation ideas, see what works, and iterate quickly.

## Architecture Overview

This is a Convex backend service that manages AI agents. The architecture
consists of:

- **Convex Functions**: All backend logic lives in the `convex/` directory as
  serverless functions
- **Database Schema**: Defined in `convex/schema.ts` with an `agents` table
  storing agent configurations
- **Agent Integration**: Communicates with an external agent worker service at
  `https://agent-worker.bluerage-software.workers.dev`

## Key Components

### Database Operations (`convex/agents.ts`)

- **Queries**: `getById`, `getAll` - Read agent data
- **Mutations**: `create`, `update`, `remove` - Modify agent records
- **Actions**: `runAgent` - Executes agents via external worker service using
  the `agents` npm package

## Development Commands

```bash
# Start local development server with hot reload
npx convex dev

# Deploy to production
npx convex deploy

# View logs
npx convex logs

# Open Convex dashboard
npx convex dashboard

# Run a specific function manually
npx convex run agents:getAll

# Export data
npx convex export

# Set environment variables
npx convex env set KEY value
```

## Environment Configuration

Required environment variables in `.env.local`:

- `CONVEX_DEPLOYMENT`: Your Convex deployment ID
- `CONVEX_URL`: Your Convex URL

## Important Patterns

1. **Function Types**: Use Convex's `query`, `mutation`, and `action` wrappers
   appropriately:
   - `query`: Read-only database operations
   - `mutation`: Database writes
   - `action`: External API calls or side effects

2. **Error Handling**: The `runAgent` action includes try-catch blocks for
   external API failures

3. **Type Safety**: Uses Convex's `v` validator for runtime type checking of
   function arguments

4. **Agent Execution Flow**:
   - Fetch agent configuration from database
   - Send request to external worker service
   - Return structured response or throw descriptive errors

# Convex guidelines

## Function guidelines

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

- HTTP endpoints are defined in `convex/http.ts` and require an `httpAction`
  decorator. For example:

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

- HTTP endpoints are always registered at the exact path you specify in the
  `path` field. For example, if you specify `/api/someRoute`, the endpoint will
  be registered at `/api/someRoute`.

### Validators

- Below is an example of an array validator:

```typescript
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
```

- Below is an example of a schema with validators that codify a discriminated
  union type:

```typescript
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
   ),
});
```

- Always use the `v.null()` validator when returning a null value. Below is an
  example query that returns a null value:

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const exampleQuery = query({
   args: {},
   returns: v.null(),
   handler: async (ctx, args) => {
      console.log("This query returns a null value");

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blueragesoftware/backend](https://github.com/blueragesoftware/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
