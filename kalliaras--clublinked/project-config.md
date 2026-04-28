---
trigger: always_on
description: Server action patterns for CRUD operations, authentication and authorization checks, error handling, and cache revalidation in Next.js Server Actions.
---


# Server Actions

## Overview

Server actions handle all server-side logic including database operations, authentication, and authorization. They are called from client components and provide a secure way to perform mutations and data fetching.

## File Structure

Server actions live in `actions.ts` files alongside their related pages:

```
example/
    page.tsx         // Server or client component
    actions.ts       // Server actions (database logic)
    _components/     // Client components that call actions
```

## Required Directives

All action files must start with:

```ts
"use server";
```

## Standard Action Pattern

Follow this structure for all server actions:

```ts
"use server";

import { revalidatePath } from "next/cache";

import { supabaseAdmin } from "@/lib/supabase/server";
import { getCurrentAgency } from "@/lib/utils/agency";
import { checkCurrentUserHasRole } from "@/lib/utils/roles-server";

/**
 * Creates a new resource with the provided data.
 */
export async function createResource(data: { name: string; slug: string }) {
  // 1. Get current context (agency, user, etc.)
  const agency = await getCurrentAgency();

  if (!agency) {
    throw new Error("Agency not found");
  }

  // 2. Authorization check
  const hasAdminRole = await checkCurrentUserHasRole("admin");

  if (!hasAdminRole) {
    throw new Error("Unauthorized: Admin access required");
  }

  // 3. Database operation
  const { error } = await supabaseAdmin.from("resources").insert({
    name: data.name,
    slug: data.slug,
    agency_id: agency.id,
  });

  if (error) {
    throw new Error(error.message);
  }

  // 4. Revalidate affected paths
  revalidatePath("/admin/resources");

  return { success: true };
}
```

## Action Best Practices

### 1. Always Add JSDoc Comments

Document what each action does and its parameters:

```ts
/**
 * Creates a new brand for the current agency.
 * @param data - Brand data including name, slug, and status
 * @throws Error if user is not authorized or agency not found
 */
export async function createBrand(data: BrandData) {
  // ...
}
```

### 2. Follow This Order

Every action should follow this sequence:

1. **Get context** - Fetch agency, user, or other contextual data
2. **Verify authentication** - Ensure user is logged in
3. **Check authorization** - Verify user has required role/permissions
4. **Validate ownership** - For updates/deletes, verify resource belongs to user's agency
5. **Perform database operation** - Execute the actual query
6. **Handle errors** - Throw descriptive errors
7. **Revalidate paths** - Update cached data with `revalidatePath()`

### 3. Error Handling

Always throw errors with descriptive messages that can be shown to users:

```ts
if (!agency) {
  throw new Error("Agency not found");
}

if (!hasAdminRole) {
  throw new Error("Unauthorized: Admin access required");
}

if (error) {
  throw new Error(error.message);
}
```

### 4. Cache Revalidation

Use `revalidatePath()` after mutations to update cached data:

```ts
// Revalidate specific paths
revalidatePath("/admin/brands");
revalidatePath(`/admin/brands/${brandId}`);

// Revalidate all routes (use sparingly)
revalidatePath("/", "layout");
```

## CRUD Operation Patterns

### Create Operations

```ts
export async function createResource(data: CreateData) {
  const agency = await getCurrentAgency();

  if (!agency) {
    throw new Error("Agency not found");
  }

  const hasAdminRole = await checkCurrentUserHasRole("admin");

  if (!hasAdminRole) {
    throw new Error("Unauthorized: Admin access required");
  }

  const { error } = await supabaseAdmin.from("resources").insert({
    ...data,
    agency_id: agency.id,
  });

  if (error) {
    throw new Error(error.message);
  }

  revalidatePath("/admin/resources");
  return { success: true };
}
```

### Read Operations

```ts
export async function getResources() {
  const agency = await getCurrentAgency();

  if (!agency) {
    throw new Error("Agency not found");
  }

  const { data, error } = await supabaseAdmin
    .from("resources")
    .select("*")
    .eq("agency_id", agency.id)
    .order("created_at", { ascending: false });

  if (error) {
    throw new Error(error.message);
  }

  return data;
}
```

### Update Operations

Always verify ownership before updating:

```ts
export async function updateResource(resourceId: string, data: UpdateData) {
  const agency = await getCurrentAgency();

  if (!agency) {
    throw new Error("Agency not found");
  }

  const hasAdminRole = await checkCurrentUserHasRole("admin");

  if (!hasAdminRole) {
    throw new Error("Unauthorized: Admin access required");
  }

  // Verify ownership
  const { data: resource, error: fetchError } = await supabaseAdmin
    .from("resources")
    .select("agency_id")
    .eq("id", resourceId)
    .single();

  if (fetchError || !resource) {
    throw new Error("Resource not found");
  }

  if (resource.agency_id !== agency.id) {
    throw new Error("Unauthorized: Resource does not belong to your agency");
  }

  // Perform update with double-check
  const { error } = await supabaseAdmin
    .from("resources")
    .update(data)
    .eq("id", resourceId)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kalliaras) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
