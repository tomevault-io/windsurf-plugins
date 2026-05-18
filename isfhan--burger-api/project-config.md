---
trigger: always_on
description: Burger API is a **Bun.js-exclusive framework** - it only works with Bun.js and cannot run on Node.js or other JavaScript runtimes. This is a modern, high-performance API framework built specifically for Bun.js that provides file-based routing, middleware architecture, Zod-based validation, automatic OpenAPI generation, and wildcard routing capabilities.
---

# Burger API - Bun.js Framework Cursor Rules

## Project Overview

Burger API is a **Bun.js-exclusive framework** - it only works with Bun.js and cannot run on Node.js or other JavaScript runtimes. This is a modern, high-performance API framework built specifically for Bun.js that provides file-based routing, middleware architecture, Zod-based validation, automatic OpenAPI generation, and wildcard routing capabilities.

## Bun.js Exclusivity

### Critical Requirements

-   **ONLY works with Bun.js**: This framework cannot run on Node.js, Deno, or browsers
-   **Bun.js runtime required**: All development and deployment must use Bun.js
-   **Bun.js version**: Ensure you are using Bun.js Latest version for optimal performance
-   **Bun-specific APIs**: Leverages Bun's native HTTP server, file system, and performance features
-   **No Node.js compatibility**: Cannot use Node.js-specific modules or APIs

### Bun.js Setup

```bash
# Install Bun.js (required)
curl -fsSL https://bun.sh/install | bash

# Install dependencies
bun install

# Run development server
bun run dev

# Build project
bun run build

# Run tests
bun test
```

## Architecture & Design Patterns

### Core Principles

-   **Bun.js native**: Built specifically for Bun's performance characteristics
-   **File-based routing**: Routes automatically discovered from directory structure
-   **Wildcard routing**: Support for `[...]` wildcard routes that match multiple path segments
-   **Middleware-first**: Global and route-specific middleware support
-   **Type safety**: Full TypeScript support with Zod validation
-   **Performance**: Optimized for Bun.js with pre-computed route handlers
-   **OpenAPI integration**: Automatic documentation generation
-   **Ecosystem middleware**: Production-ready middleware collection in `ecosystem/middlewares/` directory - in progress

### Key Components

-   `Burger` class: Main framework entry point
-   `ApiRouter`: Handles API route discovery and matching using Bun's file system
-   `PageRouter`: Handles static page routing
-   `Server`: Bun.js server wrapper leveraging `Bun.serve`
-   Middleware system: Request/response processing pipeline
-   Ecosystem: Production-ready middleware collection in `ecosystem/middlewares/` directory - in progress

## Bun.js-Specific Development

### Package Management

-   **Use Bun exclusively**: Never use npm, yarn, or pnpm
-   **Install dependencies**: `bun add <package>`
-   **Development dependencies**: `bun add -d <package>`
-   **Lock file**: Use `bun.lockb` (not package-lock.json or yarn.lock)
-   **Scripts**: Reference with `bun run <script-name>`

### HTTP Server Implementation

-   **Use Bun.serve**: Leverage Bun's native HTTP server capabilities
-   **Native fetch API**: Bun provides native fetch implementation
-   **WebSocket support**: Built-in WebSocket handling
-   **Performance**: Native performance without external frameworks

```typescript
// Burger API uses Bun.serve internally
const server = Bun.serve({
    port: 3000,
    fetch(req) {
        // Route handling logic
    },
});
```

### File System Operations

-   **Bun's file system APIs**: Use Bun's optimized file operations
-   **Route discovery**: Leverages Bun's fast file system scanning
-   **Static file serving**: Native static file handling
-   **Hot reloading**: Bun's development server capabilities

## Code Style & Conventions

### File Naming

-   API routes: `route.ts` files in directory structure
-   Dynamic routes: Use `[paramName]` folder naming
-   Wildcard routes: Use `[...]` folder naming for matching multiple segments
-   Grouping: Use `(groupName)` for route organization
-   Static pages: HTML files in page directory

### Import Organization

```typescript
// Import stuff from core
import { Server } from '@core/server.js';
import { ApiRouter } from '@core/api-router.js';

// Import utils
import { collectRoutes } from '@utils';

// Import types
import type { ServerOptions, Middleware } from '@burgerTypes';
```

### Type Definitions

-   Use `BurgerRequest<T>` for typed requests with validation
-   Use `RequestHandler` for route handlers
-   Use `Middleware` for middleware functions
-   Use `BurgerNext` for middleware return types
-   Access wildcard parameters via `req.wildcardParams`

## Route Structure & Patterns

### Basic Route Template

```typescript
// OpenAPI Metadata
export const openapi = {
    get: {
        summary: 'Get Resource',
        description: 'Description of the endpoint',
        tags: ['Resource'],
        operationId: 'getResource',
    },
};

// Validation Schemas
export const schema = {
    get: {
        query: z.object({
            search: z.string().optional(),
        }),
    },
    post: {
        body: z.object({
            name: z.string().min(1, 'Name is required'),
            price: z.number().positive('Price must be positive'),
        }),
    },
};

// Route-Specific Middleware
export const middleware: Middleware[] = [
    (req: BurgerRequest): BurgerNext => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Isfhan/burger-api](https://github.com/Isfhan/burger-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
