---
trigger: always_on
description: description: Fync API integration architecture and style guide for consistent implementation patterns
---

---
description: Fync API integration architecture and style guide for consistent implementation patterns
globs: ["src/**/*.ts", "fync/src/**/*.ts"]
alwaysApply: true
---

# Fync Architecture Style Guide

This rule defines the EXACT patterns and architecture for implementing new API integrations in the Fync package. Follow these patterns precisely to ensure consistency and maintain the unified experience.

## 🏗️ Core Architecture Principles

### 1. **Functional-First Design**
- Use `function` declarations exclusively - NO arrow functions
- All functions must be pure where possible
- Prefer immutable data patterns
- Use composition over inheritance

### 2. **TypeScript Standards**
- Use `type` declarations exclusively - NO interfaces
- All type names MUST be prefixed with `T` (e.g., `TUser`, `TApiConfig`)
- Single non-exported types in component files should be named `TProps`
- Use named exports only (except for pages/views in Next.js projects)

### 3. **Code Style**
- Self-explanatory, readable code without comments
- NO inline or block comments unless absolutely necessary for obscure syntax
- Use descriptive function and variable names instead of comments
- If you feel the need to comment, rewrite for clarity instead

## 📁 File Structure Pattern

Every API integration follows this EXACT structure:

```
src/
├── [api-name]/
│   ├── index.ts          # Main API implementation
│   ├── types.ts          # API-specific types (optional)
│   └── README.md         # Usage documentation (optional)
└── core/                 # Shared architecture (DO NOT MODIFY)
```

## 🔧 Implementation Pattern

### Step 1: Define API Constants

```typescript
const [API_NAME]_API_BASE = "https://api.example.com/v1";
```

### Step 2: Define Resources Using `defineResource`

Each logical grouping of endpoints becomes a resource:

```typescript
const userResource = defineResource({
    name: "users",
    basePath: "/users",
    methods: {
        getUser: { path: "/{id}" },
        updateUser: { path: "/{id}", method: "PUT" },
        deleteUser: { path: "/{id}", method: "DELETE" },
        createUser: { path: "", method: "POST" },
    },
});
```

#### Resource Rules:
- **name**: Plural noun (users, repos, playlists)
- **basePath**: API path prefix (starts with `/`)
- **methods**: Object with method name as key

#### Method Definition Rules:
- **path**: Endpoint path with `{param}` placeholders
- **method**: HTTP method (defaults to "GET")
- **transform**: Optional response transformation function

### Step 3: Create Resources Object

```typescript
const resources = {
    users: userResource,
    posts: postResource,
    search: searchResource,
};
```

### Step 4: Create API Builder

```typescript
const build[ApiName] = createApiBuilder({
    baseUrl: [API_NAME]_API_BASE,
    auth: { type: "bearer" as const },
    headers: {
        "Content-Type": "application/json",
        // API-specific headers
    },
});
```

#### Auth Types:
- `bearer` - Authorization: Bearer {token}
- `basic` - Authorization: Basic {encoded}
- `apikey` - X-API-Key: {key}
- `oauth2` - Authorization: Bearer {token}

### Step 5: Define Module Type

```typescript
type T[ApiName]Module = TModule<typeof resources> & {
    // Helper method signatures
    getUser: (id: string) => Promise<any>;
    searchItems: (query: string, options?: any) => Promise<any>;
    // ... other convenience methods
};
```

### Step 6: Implement Main Export Function

```typescript
export function [ApiName](config: { token: string }): T[ApiName]Module {
    const base = build[ApiName](config, resources);
    const [apiName] = base as T[ApiName]Module;

    // Implement convenience methods
    [apiName].getUser = function (id: string) {
        return base.users.getUser({ id });
    };

    [apiName].searchItems = function (query: string, options?: any) {
        return base.search.search({
            q: query,
            limit: options?.limit || 20,
            ...options,
        });
    };

    return [apiName];
}
```

## 📋 Complete Implementation Template

Use this as your starting template for any new API:

```typescript
import { createApiBuilder, defineResource, type TModule } from "../core";

const [API_NAME]_API_BASE = "https://api.example.com/v1";

// Define all resources
const exampleResource = defineResource({
    name: "examples",
    basePath: "/examples",
    methods: {
        getExample: { path: "/{id}" },
        getExamples: { path: "" },
        createExample: { path: "", method: "POST" },
        updateExample: { path: "/{id}", method: "PUT" },
        deleteExample: { path: "/{id}", method: "DELETE" },
    },
});

const resources = {
    examples: exampleResource,
};

const build[ApiName] = createApiBuilder({
    baseUrl: [API_NAME]_API_BASE,
    auth: { type: "bearer" as const },
    headers: {
        "Content-Type": "application/json",
    },
});

type T[ApiName]Module = TModule<typeof resources> & {
    // Convenience method signatures
    getExample: (id: string) => Promise<any>;
};


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/remcostoeten) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
