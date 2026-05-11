---
trigger: always_on
description: - You are the best developer in the world. But you can make mistakes, being the best means you can recognize this
---


# You
- You are the best developer in the world. But you can make mistakes, being the best means you can recognize this
- You will keep a clean and maintainable codebase

## Project info
This project only usese these:
Cloudflare Vite Plugin
Bun (never suggest npm)
Solid-js
Tanstack Solid Router, Solid Query
Tailwindcss
Solid-ui (url: https://www.solid-ui.com/) [add like this: bunx solidui-cli@latest add <component-name> ]
Vite
Hono js
Convex

## Icons
Grab icons from here https://lucide.dev/icons/ and place them in our src/components/ui/icon.tsx file

## Authentication
Better-auth with D1 and KV cloudflare [../auth.ts]

## Fetching
Always use Tanstack Query and leverate mutations where it makes sense and take advantage of all its features like optimistic updates, caching, etc. 

## Deployment
Cloudflare pages and workers

## Coding
Always follow Tanstack best practices
Always follow Solid js best practices, doing things the solid js way. Refer to their documentation when you are unsure.

## MCP
For documentation always use context7 MCP and get up to date information always.

Always look at [package.json](mdc:package.json) before recommending installing a new package.

## Design:
- clean, minimal, industrial, with consistent use of whitespace and subtle visual accent
- consistent spacing, subtle use of color, and better visual hierarchy.

- Ispiration: Apple, Linear, shadcn/ui


# Development Notes
Think carefully and only action the specific task I have given you with the most concise and elegant solution that changes as little code as possible.

## SolidJS Reactivity Guidelines ⚠️

**AVOID CIRCULAR DEPENDENCIES** - These cause "too much recursion" errors:

### ❌ Dangerous Patterns:
```javascript
// DON'T: Effect that reads query data and updates state affecting same query
createEffect(() => {
  const data = someQuery.data(); 
  setSomeState(data.map(...)); // Can trigger infinite re-renders
});

// DON'T: Memos for simple calculations
const size = createMemo(() => props.size || { width: 320, height: 384 });

// DON'T: Creating memos inside render loops
<For each={items}>
  {(item) => {
    const computed = createMemo(() => item.data); // Created on every render!
    return <div>{computed()}</div>;
  }}
</For>

// DON'T: Using function as its own default context
const context = contextObject || myFunction; // Self-reference
```

### ✅ Safe Patterns:
```javascript
// DO: Guard effects with change detection
createEffect(() => {
  const data = someQuery.data();
  if (!data || !hasActuallyChanged(data)) return;
  
  batch(() => { // Group state updates
    setSomeState(newValue);
    setOtherState(otherValue);
  });
});

// DO: Simple functions for basic calculations  
const size = () => props.size || { width: 320, height: 384 };

// DO: Create memos outside render loops
const computed = createMemo(() => items().map(item => transform(item)));

// DO: Use stable default contexts
const defaultContext = {};
const context = contextObject || defaultContext;
```

### Key Rules:
1. **Effects should not update state that affects their own dependencies**
2. **Use `batch()` to group related state updates**  
3. **Add guards to prevent unnecessary effect runs**
4. **Create memos at component level, not inside loops**
5. **Avoid self-referential contexts/defaults**

### Client-Side Authentication & State Management

User authentication state on the client is managed by TanStack Query, providing a robust, cacheable, and reactive "source of truth" for the user's session.

-   **Session Caching**: A global query with the key `['session']` is responsible for fetching and caching the user's session data. This is defined in `src/lib/auth-guard.ts` within `sessionQueryOptions`.
-   **API Client**: The `better-auth/client` library, initialized in `src/lib/auth-client.ts`, handles the low-level communication with the backend authentication API (e.g., fetching the session, signing out).
-   **Route Protection**: TanStack Router's `loader` functions are used to protect routes. The `protectedLoader` in `src/lib/auth-guard.ts` ensures that a valid session exists before rendering a protected route, redirecting to `/auth` if not.
-   **Sign-Out**: The `useSignOut` hook in `src/lib/auth-actions.ts` orchestrates the sign-out process by calling the auth client, manually clearing the `['session']` from the query cache for immediate UI updates, and redirecting the user.

This setup decouples the UI components from the authentication logic, allowing components to simply use the data from the `['session']` query to render user-specific content.

### OAuth Callback Handling in Development

When using a Single-Page Application (SPA) framework like SolidJS with Cloudflare Pages and the Vite plugin, a specific challenge arises with OAuth callbacks (e.g., from Google Sign-In) in the local development environment.

**The Problem:**

Cloudflare's SPA configuration (`"not_found_handling": "single-page-application"`) is designed to serve your `index.html` for any "navigation" request that doesn't match a static file. An OAuth redirect from a provider like Google is a navigation request.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhomra21/convex-cloudflare-workers-solid-tanstack-spa-betterauth-D1-KV](https://github.com/jhomra21/convex-cloudflare-workers-solid-tanstack-spa-betterauth-D1-KV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
