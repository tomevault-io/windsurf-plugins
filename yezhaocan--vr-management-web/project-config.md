---
trigger: always_on
description: This document explains how to implement pagination for large datasets in CloudBase document database.
---

# Pagination with CloudBase

This document explains how to implement pagination for large datasets in CloudBase document database.

## Basic Pagination Concepts

Pagination allows you to retrieve large datasets in smaller, manageable chunks (pages).

**Key Parameters:**
- `pageSize` - Number of records per page
- `pageNum` - Current page number (1-based)
- `skip()` - Number of records to skip
- `limit()` - Maximum records to return

## Simple Pagination Implementation

### Basic Page-based Query

```javascript
const pageSize = 10;  // Records per page
const pageNum = 1;    // Current page (1-based)

const result = await db.collection('todos')
    .orderBy('createdAt', 'desc')
    .skip((pageNum - 1) * pageSize)
    .limit(pageSize)
    .get();

console.log('Page', pageNum, 'data:', result.data);
```

### Calculation Formula

```javascript
// For page N:
const skip = (pageNum - 1) * pageSize;
const limit = pageSize;
```

## Complete Pagination Function

Here's a reusable pagination function:

```javascript
/**
 * Paginate through a collection
 * @param {string} collectionName - Name of the collection
 * @param {number} page - Page number (1-based)
 * @param {number} pageSize - Records per page
 * @param {object} whereConditions - Query conditions (optional)
 * @param {string} sortField - Field to sort by (optional)
 * @param {string} sortDirection - 'asc' or 'desc' (optional)
 */
async function paginateCollection(
    collectionName, 
    page = 1, 
    pageSize = 10,
    whereConditions = {},
    sortField = 'createdAt',
    sortDirection = 'desc'
) {
    const skip = (page - 1) * pageSize;
    
    let query = db.collection(collectionName);
    
    // Apply conditions if provided
    if (Object.keys(whereConditions).length > 0) {
        query = query.where(whereConditions);
    }
    
    // Apply sorting
    if (sortField) {
        query = query.orderBy(sortField, sortDirection);
    }
    
    // Apply pagination
    const result = await query
        .skip(skip)
        .limit(pageSize)
        .get();
    
    return {
        data: result.data,
        page: page,
        pageSize: pageSize,
        hasMore: result.data.length === pageSize
    };
}

// Usage
const pageData = await paginateCollection('todos', 2, 20, { status: 'active' });
console.log('Page 2 data:', pageData);
```

## Getting Total Count

To show "Page X of Y", you need the total count:

```javascript
async function paginateWithCount(collectionName, page, pageSize, whereConditions = {}) {
    const skip = (page - 1) * pageSize;
    
    // Get paginated data
    const dataQuery = db.collection(collectionName);
    const countQuery = db.collection(collectionName);
    
    if (Object.keys(whereConditions).length > 0) {
        dataQuery.where(whereConditions);
        countQuery.where(whereConditions);
    }
    
    // Execute both queries
    const [dataResult, countResult] = await Promise.all([
        dataQuery
            .orderBy('createdAt', 'desc')
            .skip(skip)
            .limit(pageSize)
            .get(),
        countQuery.count()
    ]);
    
    const totalCount = countResult.total;
    const totalPages = Math.ceil(totalCount / pageSize);
    
    return {
        data: dataResult.data,
        pagination: {
            currentPage: page,
            pageSize: pageSize,
            totalCount: totalCount,
            totalPages: totalPages,
            hasNextPage: page < totalPages,
            hasPrevPage: page > 1
        }
    };
}

// Usage
const result = await paginateWithCount('todos', 1, 10, { status: 'active' });
console.log(`Page ${result.pagination.currentPage} of ${result.pagination.totalPages}`);
console.log(`Total items: ${result.pagination.totalCount}`);
```

## Cursor-based Pagination

For real-time data or better performance, use cursor-based pagination:

```javascript
/**
 * Cursor-based pagination using a field value as cursor
 */
async function paginateWithCursor(collectionName, cursor = null, pageSize = 10) {
    const _ = db.command;
    let query = db.collection(collectionName);
    
    // If cursor exists, query records after cursor
    if (cursor) {
        query = query.where({
            createdAt: _.lt(cursor) // Assuming descending order
        });
    }
    
    const result = await query
        .orderBy('createdAt', 'desc')
        .limit(pageSize + 1) // Fetch one extra to check if more exists
        .get();
    
    const hasMore = result.data.length > pageSize;
    const data = hasMore ? result.data.slice(0, pageSize) : result.data;
    const nextCursor = hasMore ? data[data.length - 1].createdAt : null;
    
    return {
        data: data,
        nextCursor: nextCursor,
        hasMore: hasMore
    };
}

// Usage - First page
const firstPage = await paginateWithCursor('todos', null, 10);
console.log('First page:', firstPage.data);

// Next page using cursor
const secondPage = await paginateWithCursor('todos', firstPage.nextCursor, 10);
console.log('Second page:', secondPage.data);
```

## React Component Example

Here's how to implement pagination in a React component:

```javascript
import { useState, useEffect } from 'react';

function TodoList() {
    const [todos, setTodos] = useState([]);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yezhaocan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
