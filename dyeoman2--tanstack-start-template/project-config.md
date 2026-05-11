---
trigger: always_on
description: - Strong end-to-end type safety (database → server → router → client); no `any` or type casting.
---

# TanStack Start Agent Guide

## Core Philosophy

- Strong end-to-end type safety (database → server → router → client); no `any` or type casting.
- Server-first, progressively enhanced flows; UI hydrates cleanly but works without JS.
- Performance by design: static imports, parallel data fetching, targeted cache updates.

## Architecture

- `src/routes/`: File-based routes with loaders, guards, pending/error components.
- `src/features/`: Feature slices with UI, hooks, and server functions (`*.server.ts`).
- `src/lib/server/`: Server utilities (database, auth, email, env).
- `src/components/ui/`: Shadcn/ui primitives and shared components.
- Database access via Convex queries/mutations and `setupFetchClient`.

## Golden Rules

- Keep imports static and synchronous inside server modules; no dynamic imports in server functions.
- One server function, one responsibility. Compose higher-level flows by orchestrating smaller server functions.
- Use Convex queries/mutations for data operations with automatic type generation.
- Reuse provided auth guards (`routeAuthGuard`, `routeAdminGuard`, `requireAuth`, `requireAdmin`)—do not reimplement session checks.
- Keep UI components pure; business logic lives in hooks or server functions.
- Never commit files with git unless explicitly requested by the user.

## Key Patterns

### Routes

```ts
export const Route = createFileRoute('/app')({
  pendingMs: 150,
  pendingMinMs: 250,
  pendingComponent: ShellSkeleton,
  component: AppLayout,
  errorComponent: DashboardErrorBoundary,
});

function AppLayout() {
  const navigate = useNavigate();
  const location = useLocation();
  const { isAuthenticated, isPending } = useAuth();
  const redirectRef = useRef(false);
  const redirectTarget = location.href ?? '/app';

  useEffect(() => {
    if (isPending) return;

    if (!isAuthenticated) {
      if (redirectRef.current) return;

      redirectRef.current = true;
      void navigate({
        to: '/login',
        search: { redirect: redirectTarget },
        replace: true,
      }).catch(() => {
        redirectRef.current = false;
      });
    } else {
      redirectRef.current = false;
    }
  }, [isAuthenticated, isPending, navigate, redirectTarget]);

  if (isPending || !isAuthenticated) {
    return <ShellSkeleton />;
  }

  return (
    <AppChrome>
      <Outlet />
    </AppChrome>
  );
}
```

### Client Data + Convex Hooks

```ts
export function DashboardRoute() {
  const live = useQuery(api.dashboard.getDashboardData, {});

  if (live === undefined) {
    return <DashboardSkeleton />;
  }

  if (live === null) {
    return <DashboardAccessFallback />;
  }

  return <Dashboard data={live} />;
}
```

### Marketing SSG + App SPA Flow

- Public marketing/auth routes use `staticData: true` for static HTML.
- `/app` routes run as an SPA: components call Convex `useQuery` directly and render skeletons while subscriptions warm up.
- Keep layout-level guards client-side for UX, and rely on Convex server functions (`requireAuth`, `requireAdmin`) for true authorization.
- When Convex returns `null` (lost session or access), invalidate the router and prompt the user to refresh or sign back in.

### Database

- Use Convex queries/mutations with automatic type generation.
- Use `setupFetchClient` for server-side Convex operations.
- Schema defined in `convex/schema.ts` with automatic deployment.
- Real-time subscriptions available via Convex React hooks.

### Auth

- Route guards: `routeAuthGuard`, `routeAdminGuard({ location })` in `beforeLoad`.
- Server guards: `requireAuth()`, `requireAdmin()` throw on failure.
- Client auth: `useAuth()` hook from `~/features/auth/hooks/useAuth`.

### Convex Client Hooks

- Use `useQuery(api.xxx)` from `convex/react` for real-time data and show skeleton placeholders while subscriptions warm up.
- Use `useMutation(api.xxx)` for mutations with automatic cache updates.
- No manual cache invalidation needed - Convex automatically updates queries when data changes.
- Real-time subscriptions enable live data updates across all connected clients.
- **Prefer direct Convex queries over server function wrappers** - if a server function only calls a Convex query with no server-specific logic, use the Convex query directly on the client.

### When to Use Convex vs TanStack Start Server Functions

**Use Convex Queries/Mutations (Client-Side):**

- ✅ Read operations (queries) - no server-specific dependencies
- ✅ Write operations (mutations) - no server-specific dependencies
- ✅ Real-time subscriptions
- ✅ Simple database operations that don't need request context

**Use Convex Actions:**

- ✅ Need to make HTTP requests to third-party services
- ✅ Need to call LLMs or AI services
- ✅ Need to send emails
- ✅ Operations that can't be pure queries/mutations

**Use Convex HTTP Endpoints (`convex/http.ts`):**

- ✅ Expose HTTP endpoints directly from Convex
- ✅ Health checks, webhooks, public APIs
- ✅ Don't need TanStack Start server function wrapper

**Use TanStack Start Server Functions (`createServerFn`):**

- ✅ Need request context (headers, cookies, IP addresses)
- ✅ Need server-side environment variables/secrets
- ✅ Route guards (SSR requirement - `beforeLoad`)
- ✅ Complex orchestration of multiple services

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dyeoman2/tanstack-start-template](https://github.com/dyeoman2/tanstack-start-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
