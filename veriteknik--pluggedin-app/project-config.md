---
trigger: always_on
description: 1. The app uses a Next.js App Router structure:
---

# Plugged.in Project Intelligence Rules

## Project Structure Patterns

1. The app uses a Next.js App Router structure:
   - `app/` - Main application folder
   - `app/(sidebar-layout)/` - Routes with the sidebar layout
   - `app/actions/` - Server actions for backend operations
   - `app/api/` - API routes for external access
   - `db/` - Database schema and utilities
   - `types/` - TypeScript type definitions
   - `components/` - Reusable UI components
   - `hooks/` - Custom React hooks

2. MCP server logic is organized as:
   - `db/schema.ts` - Database schema definitions
   - `app/actions/mcp-servers.ts` - Server actions for MCP servers
   - `app/actions/custom-mcp-servers.ts` - Server actions for custom MCP servers
   - `types/mcp-server.ts` - Type definitions for MCP servers
   - `app/(sidebar-layout)/(container)/mcp-servers/` - UI for MCP server management

3. MCP playground components:
   - `app/actions/mcp-playground.ts` - Server actions for MCP playground
   - `app/actions/playground-settings.ts` - Server actions for playground settings
   - `app/(sidebar-layout)/(container)/mcp-playground/page.tsx` - UI implementation
   - `app/(sidebar-layout)/(container)/mcp-playground/README.md` - Documentation

## Code Patterns

1. Server-side operations use Next.js server actions:
   ```typescript
   'use server';
   
   export async function serverAction(arg1, arg2) {
     // Database operations or other server-side logic
   }
   ```

2. Database operations use Drizzle ORM with PostgreSQL:
   ```typescript
   import { db } from '@/db';
   import { someTable } from '@/db/schema';
   
   // Query example
   const result = await db.query.someTable.findFirst({
     where: eq(someTable.id, someId)
   });
   
   // Insert example
   await db.insert(someTable).values({
     field1: value1,
     field2: value2
   });
   ```

3. Frontend data fetching uses SWR:
   ```typescript
   const { data, error, mutate } = useSWR(
     key,
     () => fetchFunction(args)
   );
   ```

4. UI components follow a pattern using shadcn/ui components:
   - Use of Radix UI primitives with Tailwind styling
   - Card-based layouts for information display
   - Form components for user input
   - Dialog components for modal interactions

5. Error handling pattern for server actions:
   ```typescript
   try {
     // Operation logic
     return { success: true, data };
   } catch (error) {
     console.error('Operation failed:', error);
     return { 
       success: false, 
       error: error instanceof Error ? error.message : 'Unknown error' 
     };
   }
   ```

6. Settings management pattern:
   ```typescript
   // Define settings type
   export type SomeSettings = {
     option1: string;
     option2: number;
     option3: boolean;
   };
   
   // Get settings with default fallback
   export async function getSettings(profileUuid: string) {
     try {
       const settings = await db.query.settingsTable.findFirst({
         where: eq(settingsTable.profile_uuid, profileUuid),
       });
       
       if (!settings) {
         // Return default settings
         return {
           success: true, 
           settings: DEFAULT_SETTINGS
         };
       }
       
       return { success: true, settings: mapDbToSettings(settings) };
     } catch (error) {
       return {
         success: false,
         error: error instanceof Error ? error.message : 'Unknown error'
       };
     }
   }
   ```

## MCP Implementation Patterns

1. MCP Server Types:
   - STDIO: Command-line based servers (command + args + env)
   - SSE: HTTP-based servers (url)

2. Server Configuration Schema:
   ```typescript
   {
     name: string;
     description?: string;
     type: McpServerType; // STDIO or SSE
     command?: string; // For STDIO servers
     args?: string[]; // For STDIO servers
     env?: { [key: string]: string }; // For STDIO servers
     url?: string; // For SSE servers
     status: McpServerStatus; // ACTIVE, INACTIVE, etc.
   }
   ```

3. Custom MCP Server Pattern:
   - Python code stored in the database (codes table)
   - Additional args and env vars for execution
   - Associated with a specific profile (workspace)

4. MCP Playground Session Pattern:
   ```typescript
   // Session storage with interface
   interface McpPlaygroundSession {
     agent: ReturnType<typeof createReactAgent>;
     cleanup: McpServerCleanupFn;
     lastActive: Date;
   }
   
   const activeSessions = new Map<string, McpPlaygroundSession>();
   
   // Automatic session cleanup
   function cleanupInactiveSessions() {
     const now = new Date();
     for (const [profileUuid, session] of activeSessions.entries()) {
       if (now.getTime() - session.lastActive.getTime() > SESSION_TIMEOUT) {
         session.cleanup().catch(console.error);
         activeSessions.delete(profileUuid);
       }
     }
   }
   
   // Run cleanup at regular intervals
   setInterval(cleanupInactiveSessions, CLEANUP_INTERVAL);
   
   // Session initialization
   async function getOrCreatePlaygroundSession(profileUuid, serverUuids, llmConfig) {
     // Check for existing session
     const existingSession = activeSessions.get(profileUuid);
     if (existingSession) {
       // Update last active time
       existingSession.lastActive = new Date();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VeriTeknik/pluggedin-app](https://github.com/VeriTeknik/pluggedin-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
