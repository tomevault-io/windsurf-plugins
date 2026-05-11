---
trigger: always_on
description: Ensures React components correctly handle auth state before calling admin-protected Convex queries with Clerk authentication.
---


# Clerk + Convex Authentication Pattern

This rule ensures React components using Convex queries with Clerk authentication correctly handle loading and authentication states before attempting to fetch data, especially for admin-protected queries.

## Current Implementation Pattern

### 1. Clerk JWT Claims Configuration

**Important:** This project uses Clerk as the identity source with custom JWT claims forwarded to Convex.

Clerk JWT Template Configuration (in Clerk Dashboard):

```json
{
  "role": "{{user.public_metadata.role}}",
  "organizerGroupIds": "{{user.public_metadata.organizerGroupIds}}"
}
```

This forwards:

- `public_metadata.role` as top-level `role` claim in JWT
- `public_metadata.organizerGroupIds` as top-level `organizerGroupIds` claim in JWT

### 2. Backend: Convex Authorization Helpers

**Location:** `convex/users.ts`

#### Type-Safe Identity Access

```typescript
// Define custom identity type for type safety
type ClerkIdentity = {
  subject: string;
  email?: string;
  emailVerified?: boolean;
  username?: string;
  role?: "admin" | "manager" | "organizer";
  organizerGroupIds?: string[];
  // ... other Clerk fields
};

// Helper to get typed identity
async function getIdentityOrThrow(
  ctx: QueryCtx | MutationCtx,
): Promise<ClerkIdentity> {
  const identity = await ctx.auth.getUserIdentity();
  if (!identity) {
    throw new Error("No identity found. User not authenticated.");
  }
  return identity as ClerkIdentity;
}
```

#### Current Admin Check (Single Role)

```typescript
// Check if user is admin (boolean return)
export async function isUserAdmin(
  ctx: QueryCtx | MutationCtx,
): Promise<boolean> {
  try {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return false;

    // Access role from top-level claim
    const clerkTokenRole = (identity as any).role;
    return clerkTokenRole === "admin";
  } catch (error) {
    return false;
  }
}

// Require admin role (throws if not admin)
export async function requireAdminRole(
  ctx: QueryCtx | MutationCtx,
): Promise<void> {
  const identity = await ctx.auth.getUserIdentity();
  if (!identity) {
    throw new Error("Authentication required for admin action.");
  }

  const clerkTokenRole = (identity as any).role;
  if (clerkTokenRole !== "admin") {
    throw new Error(
      "Admin privileges required. Role 'admin' not found in Clerk token.",
    );
  }
}

// Query for frontend to check admin status
export const checkIsUserAdmin = query({
  args: {},
  returns: v.boolean(),
  handler: async (ctx) => {
    return await isUserAdmin(ctx);
  },
});
```

#### Future Multi-Role Helpers (from adminroles.md)

```typescript
// Admin or Manager access
export async function requireAdminOrManager(
  ctx: QueryCtx | MutationCtx,
): Promise<void> {
  const identity = await getIdentityOrThrow(ctx);
  const role = identity.role;

  if (role !== "admin" && role !== "manager") {
    throw new Error("Admin or Manager privileges required.");
  }
}

// Admin or Organizer for specific group
export async function requireAdminOrOrganizerForGroup(
  ctx: QueryCtx | MutationCtx,
  groupId: Id<"judgingGroups">,
): Promise<void> {
  const identity = await getIdentityOrThrow(ctx);
  const role = identity.role;

  if (role === "admin") return; // Admin has access to all groups

  if (role === "organizer") {
    const organizerGroupIds = identity.organizerGroupIds || [];
    if (organizerGroupIds.includes(groupId)) return;
  }

  throw new Error(
    "Access denied. You do not have permission for this judging group.",
  );
}
```

### 3. Frontend: React Component Auth Pattern

#### Required Pattern for Admin-Protected Components

```typescript
import { useConvexAuth, useQuery } from "convex/react";
import { api } from "../../convex/_generated/api";
import { NotFoundPage } from "../pages/NotFoundPage";

export function AdminComponent() {
  // 1. Get Convex auth state
  const { isLoading: authIsLoading, isAuthenticated } = useConvexAuth();

  // 2. Check if user is admin (skip if not authenticated)
  const isUserAdmin = useQuery(
    api.users.checkIsUserAdmin,
    isAuthenticated ? {} : "skip"
  );

  // 3. Conditionally skip admin queries
  const adminData = useQuery(
    api.adminQueries.getTotalSubmissions,
    (authIsLoading || !isAuthenticated) ? "skip" : {}
  );

  // 4. Handle auth loading state FIRST
  if (authIsLoading || (isAuthenticated && isUserAdmin === undefined)) {
    return <div>Loading authentication...</div>;
  }

  // 5. Handle unauthorized access
  if (!isAuthenticated || isUserAdmin === false) {
    return <NotFoundPage />;
  }

  // 6. Render authorized content
  return <div>Admin content: {adminData}</div>;
}
```

#### Loading State Priority Order

```typescript
// 1. Auth loading or admin check loading
if (authIsLoading || (isAuthenticated && isUserAdmin === undefined)) {
  return <div>Loading authentication...</div>;
}

// 2. Not authenticated or not admin
if (!isAuthenticated || isUserAdmin === false) {
  return <NotFoundPage />;
}

// 3. Data loading (only after auth is confirmed)
if (isAuthenticated && adminData === undefined) {
  return <div>Loading data...</div>;
}

// 4. Render content
```

### 4. Common Patterns by Component Type


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/better-todo](https://github.com/waynesutton/better-todo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
