---
trigger: always_on
description: This document provides context for AI assistants working on this codebase. It outlines the technical stack, architectural patterns, coding conventions, and project structure.
---

# AGENTS.md - Project Documentation for AI Assistants

This document provides context for AI assistants working on this codebase. It outlines the technical stack, architectural patterns, coding conventions, and project structure.

---

## Project Overview

**Name:** `@scandinavianairlines/react-router-azure-functions`  
**Type:** Server Adapter Library  
**Purpose:** Adapts Azure Functions HTTP requests to work with React Router v7 applications  
**Language:** JavaScript (ES Modules) with JSDoc for type safety  
**Runtime:** Node.js 20+  
**License:** MIT

---

## Technology Stack

### Core Dependencies

- **Runtime Platform:** Azure Functions v4 Programming Model
- **Framework Integration:** React Router v7 (`react-router`)
- **Module System:** ES Modules (`"type": "module"`)
- **Node Version:** >= 20.0.0

### Development Tools

- **Testing:** Vitest with coverage (`@vitest/coverage-v8`)
- **Type Checking:** TypeScript (`tsc --emitDeclarationOnly --checkJs`)
- **Linting:** ESLint with Neostandard config
- **Formatting:** Prettier with import sorting (`@trivago/prettier-plugin-sort-imports`)
- **Git Hooks:** Husky + lint-staged
- **Commit Convention:** Conventional Commits (Commitlint)
- **Package Manager:** Yarn 4.11.0

### Type System

- **Approach:** JSDoc comments in JavaScript files
- **Type Generation:** Automatic via `tsc --emitDeclarationOnly`
- **Output:** TypeScript declaration files (`.d.ts`) in `types/` directory
- **Validation:** `tsc --noEmit --checkJs` for type checking without compilation

---

## Architecture Patterns

### Adapter Pattern

The library implements the **Adapter Pattern** to bridge two incompatible interfaces:

```
Azure Functions HTTP ←→ Web Fetch API ←→ React Router
```

**Key Transformations:**

1. `HttpRequest` (Azure) → `Request` (Web Fetch API)
2. `Response` (Web Fetch API) → `HttpResponseInit` (Azure)
3. URL parsing from Azure-specific headers

### Separation of Concerns

The codebase is organized into distinct responsibilities:

```
┌─────────────────────────────────────────┐
│  Azure Functions HTTP Request           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  URL Parser (urlParser)                 │
│  - Extracts URL from headers            │
│  - Handles x-forwarded-host, etc.       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Request Transformer                    │
│  (createRequest)                   │
│  - Creates Web Request object           │
│  - Handles method, headers, body        │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Framework Handler                      │
│  (React Router)                         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Response Transformer                   │
│  (toAzureResponse)                      │
│  - Converts to Azure format             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Azure Functions HTTP Response          │
└─────────────────────────────────────────┘
```

### Functional Programming

- **Pure Functions:** Most functions are pure (no side effects)
- **Immutability:** No mutation of input parameters
- **Higher-Order Functions:** `createRequestHandler` returns a configured handler function
- **Composition:** Functions are composed to create the request/response pipeline

---

## Coding Conventions

### JavaScript Style

**Module System:**

```javascript
// ✅ Always use ES Modules
import { foo } from './module.js';
export function bar() {}

// ❌ Never use CommonJS
const foo = require('./module');
module.exports = bar;
```

**Function Declarations:**

```javascript
// ✅ Prefer function declarations for exported functions
export function createRequestHandler(options) {
  // ...
}

// ✅ Use arrow functions for callbacks and internal functions
const handler = createReactRouterRequestHandler(options.build, options.mode || process.env.NODE_ENV);
```

**Async/Await:**

```javascript
// ✅ Always use async/await, never raw Promises
async function handleRequest(request, context) {
  const data = await fetchData();
  return processData(data);
}

// ❌ Avoid Promise chains
function handleRequest(request, context) {
  return fetchData().then(processData);
}
```

### Type Safety with JSDoc

**All functions must have complete JSDoc:**

```javascript
/**
 * Brief description of what the function does.
 * @param {Type} paramName - Description of parameter.
 * @param {import('module').Type} [optionalParam] - Optional parameter.
 * @returns {ReturnType} Description of return value.
 */
export function functionName(paramName, optionalParam) {
  // implementation
}
```

**Import types from external modules:**

```javascript
/**
 * @typedef {(request: Request, context: import('@azure/functions').InvocationContext) => Promise<import('react-router').AppLoadContext>} GetLoadContextFn
 */
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scandinavianairlines/react-router-azure-functions](https://github.com/scandinavianairlines/react-router-azure-functions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
