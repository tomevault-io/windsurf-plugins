---
trigger: always_on
description: This document provides detailed instructions for AI agents on how to extend the Make TypeScript SDK repository in a consistent and maintainable way. The SDK follows strict patterns and conventions that must be adhered to when adding new functionality.
---

# Make TypeScript SDK - AI Agents Guide

## Overview

This document provides detailed instructions for AI agents on how to extend the Make TypeScript SDK repository in a consistent and maintainable way. The SDK follows strict patterns and conventions that must be adhered to when adding new functionality.

## Repository Structure

```
make-sdk/
├── src/
│   ├── endpoints/            # API endpoint implementations
│   │   ├── *.ts              # Endpoints
│   │   └── *.tools.ts        # Tool definitions (MCP / CLI / …)
│   ├── index.ts              # Main entry point with all exports
│   ├── make.ts               # Core Make client class
│   ├── types.ts              # Common type definitions
│   ├── utils.ts              # Utility functions
│   └── version.ts            # Auto-generated version file
├── test/
│   ├── mocks/                # JSON mock files organized by endpoint
│   ├── *.spec.ts             # Unit tests
│   ├── *.integration.test.ts # Integration tests
│   └── test.utils.ts         # Test utilities
└── dist/                     # Compiled output (auto-generated)
```

## Core Patterns and Conventions

### 1. Endpoint Implementation Pattern

**File Location**: `src/endpoints/{endpoint-name}.ts`

Every endpoint follows this exact structure:

```typescript
import type { FetchFunction, Pagination, PickColumns } from '../types.js';

/**
 * Main entity type with comprehensive JSDoc
 */
export type EntityName = {
    /** Unique identifier */
    id: number;
    /** Entity name */
    name: string;
    /** Additional fields with detailed descriptions */
    // ... other fields
};

/**
 * Options for listing entities with generic column selection
 */
export type ListEntityNamesOptions<C extends keyof EntityName = never> = {
    /** Specific columns/fields to include in the response */
    cols?: C[] | ['*'];
    /** Pagination options */
    pg?: Partial<Pagination<EntityName>>;
    /** Additional filter options specific to this endpoint */
    // ... other options
};

/**
 * Options for getting a single entity
 */
export type GetEntityNameOptions<C extends keyof EntityName = never> = {
    /** Specific columns/fields to include in the response */
    cols?: C[] | ['*'];
};

/**
 * Body for creating a new entity
 */
export type CreateEntityNameBody = {
    /** Required fields for creation */
    name: string;
    // ... other required/optional fields
};

/**
 * Body for updating an entity
 */
export type UpdateEntityNameBody = {
    /** Fields that can be updated */
    name?: string;
    // ... other updatable fields
};

/**
 * Internal response types (not exported)
 */
type ListEntityNamesResponse<C extends keyof EntityName = never> = {
    entityNames: PickColumns<EntityName, C>[];
    pg: Pagination<EntityName>;
};

type GetEntityNameResponse<C extends keyof EntityName = never> = {
    entityName: PickColumns<EntityName, C>;
};

type CreateEntityNameResponse = {
    entityName: EntityName;
};

type UpdateEntityNameResponse = {
    entityName: EntityName;
};

/**
 * Class providing methods for working with entities
 */
export class EntityNames {
    readonly #fetch: FetchFunction;

    constructor(fetch: FetchFunction) {
        this.#fetch = fetch;
    }

    /**
     * List entities with optional filtering and pagination
     */
    async list<C extends keyof EntityName = never>(
        options: ListEntityNamesOptions<C> = {},
    ): Promise<PickColumns<EntityName, C>[]> {
        const response = await this.#fetch<ListEntityNamesResponse<C>>('/entity-names', {
            query: options,
        });
        return response.entityNames;
    }

    /**
     * Get a single entity by ID
     */
    async get<C extends keyof EntityName = never>(
        id: number,
        options: GetEntityNameOptions<C> = {},
    ): Promise<PickColumns<EntityName, C>> {
        const response = await this.#fetch<GetEntityNameResponse<C>>(`/entity-names/${id}`, {
            query: options,
        });
        return response.entityName;
    }

    /**
     * Create a new entity
     */
    async create(body: CreateEntityNameBody): Promise<EntityName> {
        const response = await this.#fetch<CreateEntityNameResponse>('/entity-names', {
            method: 'POST',
            body,
        });
        return response.entityName;
    }

    /**
     * Update an existing entity
     */
    async update(id: number, body: UpdateEntityNameBody): Promise<EntityName> {
        const response = await this.#fetch<UpdateEntityNameResponse>(`/entity-names/${id}`, {
            method: 'PATCH',
            body,
        });
        return response.entityName;
    }

    /**
     * Delete an entity
     */
    async delete(id: number): Promise<void> {
        await this.#fetch(`/entity-names/${id}`, {
            method: 'DELETE',
        });
    }
}
```

### 2. Core Client Integration

**File**: `src/make.ts`

When adding a new endpoint, you must:

1. Import the endpoint class at the top
2. Add a public readonly property
3. Initialize it in the constructor
4. Add comprehensive JSDoc documentation

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [integromat/make-typescript-sdk](https://github.com/integromat/make-typescript-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
