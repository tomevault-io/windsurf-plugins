---
trigger: always_on
description: title: Data Table API Development Standards
---

---
title: Data Table API Development Standards
description: Guidelines for implementing RESTful APIs to support data tables, including CRUD operations, filtering, sorting, pagination, and response formats
glob: "src/{api,server}/**/*.{ts,js,tsx,jsx}"
alwaysApply: false
---

# Data Table API Development Standards

## Introduction

This rule defines standards for building server-side APIs that support data table components. The frontend data table requires consistent API patterns for CRUD operations, filtering, sorting, pagination, and error handling. Following these standards ensures seamless integration between the frontend table component and backend services.

## API Overview

A complete data table implementation requires the following API endpoints:

1. **List/Query Endpoint**: Fetch multiple records with filtering, sorting, and pagination
2. **Single Item Endpoint**: Get details for a specific record
3. **Create Endpoint**: Add new records
4. **Update Endpoint**: Modify existing records
5. **Delete Endpoint**: Remove records (single or batch)
6. **Batch Fetch Endpoint**: Retrieve multiple specific records by IDs

For all endpoints, we follow a consistent request/response structure using RESTful principles.

## Common Response Structure

All API responses must follow these standard structures:

### Success Response

```typescript
interface SuccessResponse<T> {
  success: true;
  data: T; // Single item or array of items
  message?: string; // Optional success message
  pagination?: {
    page: number;
    limit: number;
    total_pages: number;
    total_items: number;
  };
}
```

### Error Response

```typescript
interface ErrorResponse {
  success: false;
  error: string; // Human-readable error message
  details?: any; // Optional detailed error information
  code?: string; // Optional error code
}
```

## Endpoint Implementations

### List/Query Endpoint

**URL Pattern**: `/api/{resource}`  
**Method**: `GET`  
**Purpose**: Fetch a collection of resources with filtering, sorting, and pagination

#### Query Parameters

- `search`: Text search term
- `from_date`: Start date filter (ISO format)
- `to_date`: End date filter (ISO format)
- `sort_by`: Field to sort by
- `sort_order`: Sort direction (`asc` or `desc`)
- `page`: Page number (1-based)
- `limit`: Items per page
- Additional custom filters as needed

#### Implementation Example

```typescript
// Using Hono and Drizzle ORM with PostgreSQL
import { Hono } from "hono";
import { z } from "zod";
import { and, asc, between, count, desc, eq, ilike, or, sql } from "drizzle-orm";
import { db } from "@/db";
import { myEntityTable } from "@/db/schema/my-entity";

const router = new Hono();

// Define query parameter schema with Zod
const querySchema = z.object({
  search: z.string().optional(),
  from_date: z.string().optional(),
  to_date: z.string().optional(),
  sort_by: z.enum(["created_at", "name", "id", /* add other sortable fields */]).default("created_at"),
  sort_order: z.enum(["asc", "desc"]).default("desc"),
  page: z.coerce.number().default(1),
  limit: z.coerce.number().default(10),
});

// GET /api/my-entities
router.get("/", async (c) => {
  try {
    // Parse and validate query parameters
    const result = querySchema.safeParse(c.req.query());
    
    if (!result.success) {
      return c.json({
        success: false,
        error: "Invalid query parameters",
        details: result.error.format(),
      }, 400);
    }
    
    const { search, from_date, to_date, sort_by, sort_order, page, limit } = result.data;
    
    // Build filters
    const filters = [];
    
    // Search filter (search across multiple fields)
    if (search) {
      filters.push(
        or(
          ilike(myEntityTable.name, `%${search}%`),
          ilike(myEntityTable.description, `%${search}%`),
          // Add other searchable fields
        )
      );
    }
    
    // Date filtering
    if (from_date && to_date) {
      filters.push(
        between(
          myEntityTable.created_at,
          new Date(from_date),
          new Date(to_date)
        )
      );
    } else if (from_date) {
      const fromDateTime = new Date(from_date);
      filters.push(sql`${myEntityTable.created_at} >= ${fromDateTime}`);
    } else if (to_date) {
      const toDateTime = new Date(to_date);
      filters.push(sql`${myEntityTable.created_at} <= ${toDateTime}`);
    }
    
    // Add any custom filters here
    
    // Get data with pagination
    const data = await db
      .select()
      .from(myEntityTable)
      .where(filters.length > 0 ? and(...filters) : undefined)
      .orderBy(
        sort_by === "name"
          ? sort_order === "asc" ? asc(myEntityTable.name) : desc(myEntityTable.name)
          : sort_order === "asc" ? asc(myEntityTable.created_at) : desc(myEntityTable.created_at)
        // Add other sort fields as needed
      )
      .limit(limit)
      .offset((page - 1) * limit);
    
    // Count total items (for pagination)
    const [{ value: totalItems }] = await db
      .select({ value: count() })
      .from(myEntityTable)
      .where(filters.length > 0 ? and(...filters) : undefined);
    
    return c.json({
      success: true,
      data: data,
      pagination: {
        page,
        limit,
        total_pages: Math.ceil(totalItems / limit),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacksonkasi1/tnks-data-table](https://github.com/jacksonkasi1/tnks-data-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
