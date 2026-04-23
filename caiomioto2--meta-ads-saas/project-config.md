---
trigger: always_on
description: How to add new native apps with views, MCP tools, and AI chat integration
---

# Adding Native Apps & Views with MCP Tool Integration

This guide covers the complete pattern for adding new native applications to deco CMS, from MCP tools to frontend views with AI chat integration.

## Overview

Native apps are first-class features in deco CMS that:
- Expose MCP tools for programmatic access
- Provide dedicated UI views for user interaction
- Integrate with AI chat for conversational interfaces
- Appear in sidebar navigation automatically
- Support organization and project-level scoping

## Step 1: Create MCP Tools

### 1.1 Define Tool Group

Create a new tool group that will become a virtual integration:

```typescript
// packages/sdk/src/mcp/{feature}/api.ts
import { createToolGroup } from "../context.ts";

export const createTool = createToolGroup("{Feature}", {
  name: "{Feature} Management",
  description: "Manage {feature} settings and configuration.",
  icon: "https://assets.decocache.com/mcp/{uuid}/{feature}.png",
  // workspace: true  // Only if this is project-scoped, omit for org-level
});
```

**Important**: The group name becomes the integration ID as `i:{name-in-kebab-case}`.

### 1.2 Implement Tools with Proper Return Types

MCP tools must return **objects**, not nullable primitives:

```typescript
// ❌ BAD - Returns nullable primitive
export const getTheme = createTool({
  name: "THEME_GET",
  handler: async (props, c) => {
    return theme || null;  // Type error: null not assignable to object
  },
});

// ✅ GOOD - Returns object wrapper
export const getTheme = createTool({
  name: "THEME_GET",
  outputSchema: z.object({
    theme: themeSchema.nullable(),
  }),
  handler: async (props, c) => {
    return { theme: theme || null };
  },
});
```

### 1.3 Handle Organization Context

For org-level features, auto-resolve org ID from context:

```typescript
import { getOrgIdFromContext } from "../projects/util.ts";

export const updateOrgSetting = createTool({
  name: "SETTING_UPDATE_ORG",
  inputSchema: z.object({
    // Do NOT include orgId in input - always resolve from context
    setting: settingSchema,
  }),
  handler: async (props, c) => {
    // Auto-resolve from context
    const orgId = await getOrgIdFromContext(c);
    
    if (!orgId) {
      throw new Error("No organization context available");
    }

    // Use org slug for authorization (not ID!)
    const orgSlug = c.locator?.org;
    await assertTeamResourceAccess("TEAMS_UPDATE", orgSlug, c);

    // Use drizzle for database operations
    const result = await c.drizzle
      .update(organizations)
      .set({ setting: props.setting })
      .where(eq(organizations.id, orgId))
      .returning();

    return { setting: result[0].setting };
  },
});
```

**Key Points**:
- Do NOT accept `orgId` from input - always auto-resolve from context
- Use `assertTeamResourceAccess` with **org slug**, not org ID
- Authorization system expects slugs for team resources

### 1.4 Register Tools

Add to appropriate tool collection:

```typescript
// packages/sdk/src/mcp/index.ts

// Project-scoped tools
export const PROJECT_TOOLS = [
  // ... existing tools
  
  // Project-scoped features
  featureAPI.getSetting,
  featureAPI.updateSetting,
  
  // Org-level features that need project context
  featureAPI.getOrgSetting,
  featureAPI.updateOrgSetting,
] as const;

// OR for org-level tools (if they don't need project context)
export const ORG_TOOLS = [
  // ... existing tools
  featureAPI.getOrgSetting,
  featureAPI.updateOrgSetting,
] as const;
```

## Step 2: Create CRUD Layer

### 2.1 Define Typed Operations

```typescript
// packages/sdk/src/crud/{feature}.ts
import { MCPClient } from "../fetcher.ts";
import type { ProjectLocator } from "../locator.ts";

export interface GetOrgSettingInput {
  locator: ProjectLocator;  // Always include for scoped clients
  orgId: number;
}

export const getOrgSetting = async (
  input: GetOrgSettingInput,
  init?: RequestInit,
): Promise<Setting | null> => {
  const result = await MCPClient.forLocator(input.locator).SETTING_GET_ORG(
    { orgId: input.orgId },
    init,
  );
  return (result as { setting: Setting | null }).setting;
};
```

**Key Pattern**: Always unwrap MCP tool responses that return object wrappers.

### 2.2 Create React Query Hooks

```typescript
// packages/sdk/src/hooks/{feature}.ts
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { useSDK } from "./store.tsx";

export function useOrgSetting(orgId?: number) {
  const { locator } = useSDK();

  return useQuery({
    queryKey: ["org-setting", orgId],
    queryFn: () => orgId && locator ? getOrgSetting({ locator, orgId }) : null,
    enabled: !!orgId && !!locator,
  });
}

export function useUpdateOrgSetting() {
  const client = useQueryClient();
  const { locator } = useSDK();

  return useMutation({
    mutationFn: (input: Omit<UpdateOrgSettingInput, 'locator'>) => {
      if (!locator) throw new Error('No locator available');
      return updateOrgSetting({ ...input, locator });
    },
    onSuccess: (_, variables) => {
      client.invalidateQueries({ queryKey: ["org-setting", variables.orgId] });
      
      // Invalidate related caches
      if (locator) {
        const { org } = Locator.parse(locator);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caiomioto2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
