---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Tech Stack

- **Next.js 16.0.1** (App Router with Turbopack) - React 19.2.0
- **Convex** - Real-time backend database and functions
- **Better Auth** - Authentication system with email verification, 2FA, magic links, and OAuth
- **TypeScript** - Full type safety throughout
- **Tailwind CSS v4** - Styling with dark mode support
- **Radix UI** - Accessible component primitives
- **pnpm** - Package manager

## Development Commands

### Starting the Application
```bash
# Start both Convex backend and Next.js frontend (recommended)
pnpm dev

# Start only frontend (requires Convex to be running separately)
pnpm dev:frontend

# Start only Convex backend
pnpm dev:backend
# or run once and exit:
pnpm convex dev --once
```

### Building and Deployment
```bash
# Production build
pnpm build

# Start production server
pnpm start

# Lint
pnpm lint

# Deploy Convex to production
pnpm convex deploy
```

### Convex Environment Management
```bash
# Set development environment variables
pnpm convex env set VARIABLE_NAME value

# Set production environment variables
pnpm convex env set VARIABLE_NAME value --prod

# List environment variables
pnpm convex env list
```

## Convex MCP Tools (AI Assistant Integration)

This project has the **Convex MCP server** installed, providing AI assistants with direct access to the Convex backend. See `@convex-mcp.md` for details.

### Available MCP Tools

**Prefer using MCP tools over bash commands when working with Convex** - they provide structured data and better error handling.

- **Deployment**: Use `status` tool to select your deployment
- **Tables**: Use `tables` tool to view schemas, `data` tool to browse table contents, `runOneoffQuery` for custom read-only queries
- **Functions**: Use `functionSpec` to see available functions, `run` to execute them, `logs` to view execution logs
- **Environment**: Use `envList`, `envGet`, `envSet`, `envRemove` instead of bash `convex env` commands

These tools provide structured access to Convex data and are more reliable than parsing CLI output.

## Architecture Overview

### Authentication Flow

This application uses a **dual-system authentication architecture**:

1. **Better Auth** (`src/lib/auth.ts`) - Server-side auth configuration running on Convex
   - Configures providers (Google, GitHub, Slack), email verification, password reset
   - Defines all auth options including 2FA, magic links, email OTP
   - Exports `createAuth(ctx)` which must receive Convex context

2. **Better Auth Client** (`src/lib/auth-client.ts`) - Client-side auth instance
   - React hooks and client methods for auth operations
   - Exports `authClient` used in components for sign in/out, session management
   - Plugins must match server-side configuration

3. **Convex Auth Component** (`convex/auth.ts`) - Connects Better Auth to Convex database
   - `betterAuthComponent` - handles database operations for auth
   - `onCreateUser` - creates application user record when auth user is created
   - `onDeleteUser` - cascade deletes user data (todos, etc.)
   - `onUpdateUser` - keeps email field synced between auth and app user tables
   - `getCurrentUser` - merges Better Auth user metadata with application user data

4. **HTTP Routes** (`convex/http.ts`) - Registers Better Auth API endpoints
   - Must call `betterAuthComponent.registerRoutes(http, createAuth)`
   - Handles `/api/auth/*` endpoints through Convex

5. **Proxy Protection** (`src/proxy.ts`) - Route protection middleware (renamed from middleware.ts in Next.js 16)
   - Uses cookie-based session checking (recommended approach)
   - Redirects unauthenticated users to `/sign-in`
   - Redirects authenticated users from auth pages to `/dashboard`
   - Matcher excludes static assets, `_next`, and `api/auth` routes

### Key Authentication Concepts

- **Session checking**: `proxy.ts` uses `getSessionCookie()` for performance (recommended over fetching full session)
- **User data split**: Auth metadata (email, name, image) lives in Better Auth tables; application data lives in `users` table
- **Lifecycle hooks**: `onCreateUser`, `onDeleteUser`, `onUpdateUser` keep application user table in sync with auth system
- **Plugin synchronization**: Client plugins (`src/lib/auth-client.ts`) must match server plugins (`src/lib/auth.ts`)

### Convex Integration

**Client Setup** (`src/app/ConvexClientProvider.tsx`):
- `ConvexReactClient` - connects to Convex backend using `NEXT_PUBLIC_CONVEX_URL`
- `ConvexBetterAuthProvider` - wraps app with both Convex and auth context
- Must wrap all client components that use `useQuery`, `useMutation`, or auth hooks

**Database Schema** (`convex/schema.ts`):
- Application tables defined here (e.g., `users`, `todos`)
- Better Auth tables auto-generated by `@convex-dev/better-auth`
- Indexes required for efficient queries (e.g., `userId` index on todos)

**Querying Data**:
```typescript
// In client components
import { useQuery, useMutation } from "convex/react";
import { api } from "@/convex/_generated/api";

const data = useQuery(api.moduleName.functionName, { args });
const mutate = useMutation(api.moduleName.functionName);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [podalls97/next-convex-betterauth-template](https://github.com/podalls97/next-convex-betterauth-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
