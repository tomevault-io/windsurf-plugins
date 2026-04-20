---
trigger: always_on
description: Tools are functions that the MCP (Model Context Protocol) can call. They represent discrete operations that can be executed independently or as part of workflows.
---

# Deco MCP Tools Development Guide

## Tool Architecture

Tools are functions that the MCP (Model Context Protocol) can call. They represent discrete operations that can be executed independently or as part of workflows.

## Basic Tool Pattern

```typescript
// server/main.ts structure
import { withRuntime } from "@deco/workers-runtime";
import { createTool } from "@deco/workers-runtime";
import { z } from "zod";

// Define tools (functions the MCP can call)
const createMyTool = (env: Env) => createTool({
  id: "MY_TOOL",
  description: "Tool description",
  inputSchema: z.object({ /* input schema */ }),
  outputSchema: z.object({ /* output schema */ }),
  execute: async ({ context }) => {
    // Tool logic here
    return { /* result */ };
  },
});

// Setup runtime with tools
const { Workflow, ...runtime } = withRuntime<Env>({
  workflows: [/* workflows */],
  tools: [createMyTool],
  fetch: fallbackToView("/"), // Serves frontend assets
});

export { Workflow };
export default runtime;
```

## Accessing Request Headers (Cloudflare Workers)

When you need to extract headers from the incoming request (e.g., GitHub webhooks, authentication headers), use the `runtimeContext` parameter:

```typescript
const createWebhookTool = (env: Env) => createTool({
  id: "PROCESS_WEBHOOK",
  description: "Process incoming webhook with headers",
  inputSchema: z.object({
    payload: z.any(),
  }),
  outputSchema: z.object({
    success: z.boolean(),
    headers: z.record(z.string()),
  }),
  execute: async ({ context, runtimeContext }) => {
    // Extract the request from runtime context
    const req = runtimeContext.get("req") as Request | undefined;

    if (!req) {
      throw new Error("Request not found to access headers");
    }

    // Extract headers from the request (Cloudflare Workers compatible)
    const deliveryId = req.headers.get("x-github-delivery");
    const eventType = req.headers.get("x-github-event") || "unknown";
    const signature = req.headers.get("x-hub-signature-256");
    const hookId = req.headers.get("x-github-hook-id");
    const contentType = req.headers.get("content-type");
    const userAgent = req.headers.get("user-agent");

    // Process the webhook with header information
    console.log("Webhook received:", {
      deliveryId,
      eventType,
      signature,
      hookId,
    });

    // Your webhook processing logic here
    const result = await processWebhook(context.payload, {
      eventType,
      signature,
      // ... other header data
    });

    return {
      success: true,
      headers: {
        deliveryId: deliveryId || "",
        eventType,
        signature: signature || "",
        hookId: hookId || "",
      },
    };
  },
});
```

### Common Header Patterns

```typescript
// Authentication headers
const authToken = req.headers.get("authorization");
const apiKey = req.headers.get("x-api-key");

// Request metadata
const contentType = req.headers.get("content-type");
const userAgent = req.headers.get("user-agent");
const origin = req.headers.get("origin");
const referer = req.headers.get("referer");

// Custom headers
const customHeader = req.headers.get("x-custom-header");

// IP and forwarding headers (Cloudflare specific)
const cfConnectingIp = req.headers.get("cf-connecting-ip");
const xForwardedFor = req.headers.get("x-forwarded-for");
const xRealIp = req.headers.get("x-real-ip");
```

## Database Operations in Tools

### Database Connection Pattern

**🔥 CRITICAL: Always use `getDb(env)` function**

```typescript
import { getDb } from "./db.ts";
import { eq } from "drizzle-orm";
import { myTable } from "./schema.ts";

// ✅ CORRECT: Always use getDb(env) in tools
export const createMyDatabaseTool = (env: Env) =>
  createTool({
    id: "MY_DATABASE_TOOL",
    description: "Interact with database",
    inputSchema: z.object({ /* input schema */ }),
    outputSchema: z.object({ /* output schema */ }),
    execute: async ({ context }) => {
      // ALWAYS get database connection this way
      const db = await getDb(env);
      
      // Perform database operations
      const result = await db.select().from(myTable);
      return { result };
    },
  });
```

**❌ NEVER do direct database connections:**
```typescript
// DON'T DO THIS - Never create direct connections
const db = drizzle(env);  // ❌ Missing migrations
const db = new Database(); // ❌ Wrong approach
```

### Database Operations Patterns

**1. Create (Insert):**
```typescript
execute: async ({ context }) => {
  const db = await getDb(env);
  
  const newRecord = await db.insert(myTable).values({
    title: context.title,
    completed: 0,
  }).returning({ id: myTable.id });
  
  return { id: newRecord[0].id };
}
```

**2. Read (Select):**
```typescript
execute: async ({ context }) => {
  const db = await getDb(env);
  
  // Select all
  const allRecords = await db.select().from(myTable);
  
  // Select with conditions
  const filteredRecords = await db.select()
    .from(myTable)
    .where(eq(myTable.completed, 1));
  
  // Select single record
  const singleRecord = await db.select()
    .from(myTable)
    .where(eq(myTable.id, context.id))
    .limit(1);
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deco-cx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
