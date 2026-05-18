---
trigger: always_on
description: enableSorting: false,
---

---
title: Frontend Data Table Implementation Standards
description: Guidelines for implementing data tables in React applications, including configuration, API integration, state management, and UI customization
glob: "src/{components,features,pages}/**/{table,data-table}*.{ts,tsx,js,jsx}"
alwaysApply: false
---

# Frontend Data Table Implementation Standards

## Introduction

This rule defines standards for implementing data tables in our frontend React applications. These standards ensure consistent user experience, proper API integration, and maintainable code across our applications. All data table implementations should follow these guidelines to maintain consistency and provide a robust user experience.

## Data Table Component Overview

Our advanced data table component provides the following features:

1. **Server-side data fetching**: Integration with backend APIs
2. **Pagination**: Navigate through large datasets
3. **Sorting**: Order data by specific columns
4. **Filtering**: Custom filtering including search and date ranges
5. **Row selection**: Select single or multiple rows
6. **Column visibility**: Toggle column visibility
7. **Export functionality**: Export data to CSV or Excel
8. **Row actions**: Perform operations on individual rows
9. **Bulk actions**: Perform operations on multiple selected rows
10. **URL state persistence**: Maintain table state in URL parameters

## Basic Usage Pattern

### 1. Define Your Schema

Start by defining the schema for your data using Zod:

```typescript
import { z } from "zod";

export const entitySchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email().optional(),
  created_at: z.string(),
  status: z.enum(["active", "inactive", "pending"]),
  // Add other fields as needed
});

export type Entity = z.infer<typeof entitySchema>;

export const entitiesResponseSchema = z.object({
  success: z.boolean(),
  data: z.array(entitySchema),
  pagination: z.object({
    page: z.number(),
    limit: z.number(),
    total_pages: z.number(),
    total_items: z.number(),
  }),
});
```

### 2. Create API Functions

Implement API functions to communicate with your backend:

```typescript
// src/api/entity/fetch-entities.ts
import { entitiesResponseSchema } from "@/schemas/entity-schema";

const API_BASE_URL = "/api";

export async function fetchEntities({
  search = "",
  from_date = "",
  to_date = "",
  sort_by = "created_at",
  sort_order = "desc",
  page = 1,
  limit = 10,
}) {
  // Build query parameters
  const params = new URLSearchParams();
  if (search) params.append("search", search);
  if (from_date) params.append("from_date", from_date);
  if (to_date) params.append("to_date", to_date);
  params.append("sort_by", sort_by);
  params.append("sort_order", sort_order);
  params.append("page", page.toString());
  params.append("limit", limit.toString());

  // Fetch data
  const response = await fetch(`${API_BASE_URL}/entities?${params.toString()}`);
  
  if (!response.ok) {
    throw new Error(`Failed to fetch entities: ${response.statusText}`);
  }
  
  const data = await response.json();
  return entitiesResponseSchema.parse(data);
}

export async function fetchEntitiesByIds(ids: number[]) {
  if (ids.length === 0) {
    return [];
  }
  
  // Use batching for efficiency
  const BATCH_SIZE = 50;
  const results = [];
  
  // Process in batches
  for (let i = 0; i < ids.length; i += BATCH_SIZE) {
    const batchIds = ids.slice(i, i + BATCH_SIZE);
    const params = new URLSearchParams();
    
    // Add each ID as a parameter
    batchIds.forEach(id => {
      params.append("ids", id.toString());
    });
    
    const response = await fetch(`${API_BASE_URL}/entities/batch?${params.toString()}`);
    
    if (!response.ok) {
      throw new Error(`Failed to fetch entities batch: ${response.statusText}`);
    }
    
    const data = await response.json();
    if (data.success && Array.isArray(data.data)) {
      results.push(...data.data);
    }
  }
  
  return results;
}
```

### 3. Create a Data Fetching Hook

Create a custom hook to handle data fetching with React Query:

```typescript
// src/features/entity-table/utils/data-fetching.ts
import { useQuery, keepPreviousData } from "@tanstack/react-query";
import { fetchEntities } from "@/api/entity/fetch-entities";
import { preprocessSearch } from "@/components/data-table/utils";

export function useEntitiesData(
  page: number,
  pageSize: number,
  search: string,
  dateRange: { from_date: string; to_date: string },
  sortBy: string,
  sortOrder: string
) {
  return useQuery({
    queryKey: [
      "entities",
      page,
      pageSize,
      preprocessSearch(search),
      dateRange,
      sortBy,
      sortOrder,
    ],
    queryFn: () =>
      fetchEntities({
        page,
        limit: pageSize,
        search: preprocessSearch(search),
        from_date: dateRange.from_date,
        to_date: dateRange.to_date,
        sort_by: sortBy,
        sort_order: sortOrder,
      }),
    placeholderData: keepPreviousData,
  });
}

// Add this property for the DataTable component
useEntitiesData.isQueryHook = true;
```

### 4. Define Table Columns

Define the columns for your data table:

```typescript
// src/features/entity-table/components/columns.tsx
import { format } from "date-fns";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacksonkasi1/tnks-data-table](https://github.com/jacksonkasi1/tnks-data-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
