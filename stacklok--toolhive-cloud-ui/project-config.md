---
trigger: always_on
description: This is a **Next.js 16 App Router** application for visualizing MCP (Model Context Protocol) servers running in user infrastructure. It's an **open-source** project using **TypeScript**, **React 19**, **Tailwind CSS 4**, and **Better Auth** for OIDC authentication.
---

# GitHub Copilot Instructions - ToolHive Cloud UI

## Project Context

This is a **Next.js 16 App Router** application for visualizing MCP (Model Context Protocol) servers running in user infrastructure. It's an **open-source** project using **TypeScript**, **React 19**, **Tailwind CSS 4**, and **Better Auth** for OIDC authentication.

## Core Architecture Principles

- **Server Components by default** - Use Client Components (`'use client'`) only when needed (event handlers, browser APIs, state hooks)
- **🚫 NEVER USE `any` TYPE** - STRICTLY FORBIDDEN. Use `unknown` with type guards or proper types
- **Stateless authentication** - JWT-based sessions, no server-side storage
- **hey-api generated client** - Never create manual fetch logic, always use generated hooks
- **async/await over promises** - No `.then()` chains, use async/await for readability

## ⚠️ Next.js App Router - Validate Before Suggesting

**This is Next.js 16 with App Router** - Before suggesting code, verify against [Next.js Documentation](https://nextjs.org/docs):

**Common mistakes to flag**:

- ❌ Pages Router patterns (getServerSideProps, getStaticProps, \_app.js, \_document.js)
- ❌ Custom routing logic (use file-system routing: `page.tsx`, `layout.tsx`)
- ❌ Missing `'use client'` when using hooks, events, or browser APIs
- ❌ Adding `'use client'` to Server Components unnecessarily
- ❌ Not understanding what Server Components can/cannot do
- ❌ Ignoring Next.js caching (`next: { revalidate, tags }`)

**Key App Router concepts**:

- **File conventions**: `page.tsx` (route), `layout.tsx` (shared UI), `loading.tsx` (loading state), `error.tsx` (error boundary)
- **Server Components** (default): No hooks, no events, can fetch data directly with async/await
- **Client Components** (`'use client'`): Use when you need hooks, events, browser APIs
- **Server Actions** (`'use server'`): For mutations, better than API routes for simple cases
- **Caching**: Use `revalidatePath()`, `revalidateTag()` for on-demand cache updates

## Code Review Focus Areas

### 1. Component Structure

**GOOD:**

- Server Components for data fetching
- Client Components marked with `'use client'` only when necessary
- Small, focused components with single responsibility
- Proper TypeScript interfaces for props

**BAD:**

- `'use client'` on every component
- Large monolithic components
- Missing or weak TypeScript types
- Using `any` type

### 2. Data Fetching

**GOOD:**

```typescript
// Server Component
async function ServerList() {
  const response = await fetch('/api/servers', {
    next: { revalidate: 3600 }
  });
  const data = await response.json();
  return <div>{data.map(...)}</div>;
}

// Client Component with hey-api
'use client';
import { useGetApiV0Servers } from '@/generated/client/@tanstack/react-query.gen';

function ServerList() {
  const { data, isLoading, error } = useGetApiV0Servers();
  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  return <div>{data?.map(...)}</div>;
}
```

**BAD:**

```typescript
// Manual fetch with useState/useEffect
const [data, setData] = useState(null);
useEffect(() => {
  fetch("/api/servers")
    .then((res) => res.json())
    .then(setData);
}, []);

// Using .then() instead of async/await
fetch("/api/servers").then((res) => res.json());
```

### 3. API Integration

**ALWAYS:**

- Use hey-api generated hooks from `@/generated/client/@tanstack/react-query.gen`
- Use `async/await` syntax, never `.then()`
- Handle loading and error states
- Regenerate client when API changes: `pnpm generate-client`

**NEVER:**

- Create manual fetch calls in components
- Edit files in `src/generated/` directory (auto-generated)
- Use `.then()` for promise handling
- Ignore TypeScript errors from generated types

### 4. UI Components

**ALWAYS:**

- Use shadcn/ui components from `@/components/ui/*`
- Prefer composition over complex props
- Follow Tailwind CSS 4 conventions

**NEVER:**

- Create custom Button, Dialog, Card components
- Duplicate inline styles (extract to reusable component)
- Use inline event handlers for complex logic

### 5. Authentication

**GOOD:**

- Check auth status on protected routes
- Use Better Auth hooks for client-side auth state
- Server Actions for authenticated mutations
- Graceful handling of unauthenticated users

**BAD:**

- Hardcoded auth checks
- Exposing auth tokens in client code
- Missing authentication on protected routes

### 6. TypeScript Best Practices

**🚫 CRITICAL: NEVER USE `any` TYPE - This is a code review blocker.**

**GOOD:**

```typescript
// Use 'as const' instead of enums
const Status = {
  Active: "active",
  Inactive: "inactive",
} as const;

type Status = (typeof Status)[keyof typeof Status];

// Type guards for runtime validation with unknown
function isServer(value: unknown): value is Server {
  return typeof value === "object" && value != null && "name" in value;
}

function processServer(data: unknown) {
  if (isServer(data)) {
    // Type-safe usage
    return data.name;
  }
  throw new Error("Invalid server data");
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stacklok/toolhive-cloud-ui](https://github.com/stacklok/toolhive-cloud-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
