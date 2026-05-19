---
trigger: always_on
description: compoder generate:sql-api
---

# MongoDB API Generation Guide

This guide is for generating API endpoints based on MongoDB.

## User Input

Please provide the following information:

1. MongoDB Schema definition, including:
   - Data structure and its types
   - Required fields
   - Default values
   - Validation rules

For example:

```typescript
// In lib/db/[modelName]/schema.ts
import mongoose, { Schema, model } from "mongoose"
import { DataType, ItemType } from "./types"

// Subdocument Schema
const ItemSchema = new Schema({
  value: {
    type: String,
    required: true,
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
})

// Main document Schema
const DataSchema = new Schema(
  {
    field1: {
      type: String,
      required: true,
    },
    field2: {
      type: String,
      required: true,
    },
    items: {
      type: [ItemSchema],
      required: true,
      default: [],
    },
  },
  {
    timestamps: true,
  },
)

// Export Model
export const Model = mongoose.models.Data || model<DataType>("Data", DataSchema)

// Define corresponding types in types.ts
export interface ItemType {
  _id: mongoose.Types.ObjectId
  value: string
  createdAt: Date
}

export interface DataType {
  _id: mongoose.Types.ObjectId
  field1: string
  field2: string
  items: ItemType[]
  createdAt: Date
  updatedAt: Date
}
```

2. API request type definition, including:
   - Request method (GET/POST/PUT/DELETE)
   - Request parameters and their types
   - Response data structure (optional)

For example:

```typescript
// In app/api/[modelName]/type.d.ts
declare namespace ApiNamespace {
  export interface RequestType {
    param1: string // Parameter 1 description
    param2: string // Parameter 2 description
    param3: string // Parameter 3 description
  }
}
```

## Generation Steps

1. Implement database operations (query or mutation) based on user input type definitions and MongoDB Schema
2. Create corresponding API route handlers

## Detailed Steps

### 1. Implement Database Operations

Based on the API request method, implement corresponding operations in the same directory level as the Schema file:

#### Query Operations (lib/db/[modelName]/selectors.ts)

```typescript
import { Model } from "./schema"
import { FilterQuery } from "mongoose"
import { DataType } from "./types"

export async function queryOperation({
  param1,
  param2,
  page,
  pageSize,
}: {
  param1: string
  param2: string
  page: number
  pageSize: number
}) {
  try {
    const skip = (page - 1) * pageSize

    // 1. Build query conditions
    let searchQuery: FilterQuery<DataType> = {
      field1: param1,
    }

    if (param2) {
      searchQuery.field2 = {
        $regex: param2,
        $options: "i",
      }
    }

    // 2. Execute query
    const [data, total] = await Promise.all([
      Model.find(searchQuery)
        .select("field1 field2 field3")
        .sort({ createdAt: -1 })
        .skip(skip)
        .limit(pageSize)
        .lean(),
      Model.countDocuments(searchQuery),
    ])

    // 3. Process return data
    const formattedData = data.map(item => ({
      id: item._id,
      value: item.field1,
      extra: item.field2,
    }))

    return {
      data: formattedData,
      total,
    }
  } catch (error) {
    console.error("Error in query operation:", error)
    throw error
  }
}
```

#### Mutation Operations (lib/db/[modelName]/mutations.ts)

```typescript
import { Model } from "./schema"

export async function databaseOperation({
  param1,
  param2,
  param3,
}: {
  param1: string
  param2: string
  param3: string
}) {
  try {
    // 1. Find record
    const record = await Model.findById(param1)
    if (!record) {
      throw new Error("Record not found")
    }

    // 2. Execute update operation
    const targetIndex = record.items.findIndex(
      (item: ItemType) => item._id.toString() === param2,
    )
    if (targetIndex === -1) {
      throw new Error("Target not found")
    }

    record.items[targetIndex].value = param3
    await record.save()

    // 3. Return result
    return {
      _id: record._id,
      ...record.toObject(),
    }
  } catch (error) {
    console.error("Error in database operation:", error)
    throw error
  }
}
```

### 2. Create API Route Handlers

In the same directory level as the API type definition, implement corresponding routes based on operation type:

#### List Query (app/api/[modelName]/list/route.ts)

```typescript
import { NextResponse } from "next/server"
import { queryOperation } from "@/lib/db/[modelName]/selectors"
import type { ApiNamespace } from "../type"
import { validateSession } from "@/lib/auth/middleware"
import { connectToDatabase } from "@/lib/db/mongo"

export async function GET(request: Request) {
  try {
    // 1. Validate session
    const authError = await validateSession()
    if (authError) {
      return authError
    }

    // 2. Connect to database
    await connectToDatabase()

    // 3. Get query parameters from URL
    const { searchParams } = new URL(request.url)
    const param1 = searchParams.get("param1")
    const param2 = searchParams.get("param2")
    const page = parseInt(searchParams.get("page") || "1")
    const pageSize = parseInt(searchParams.get("pageSize") || "10")

    if (!param1) {
      return NextResponse.json(
        { error: "Missing required parameter: param1" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IamLiuLv/compoder](https://github.com/IamLiuLv/compoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
