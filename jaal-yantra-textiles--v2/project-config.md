---
trigger: always_on
description: This document provides guidelines for AI agents working on this codebase.
---

# AGENTS.md

This document provides guidelines for AI agents working on this codebase.

## Project Overview

This is a **JYT Commerce API** - a Medusa 2.x-based e-commerce platform extended for textile production. Built with TypeScript, it includes:
- Admin API with 150+ endpoints for managing designs, production, partners, inventory
- AI-powered features (image generation, query planning, RAG)
- Partner portal and storefront applications
- Workflows for business processes (production runs, publishing campaigns)

## Build Commands

```bash
# Development
yarn dev                    # Start Medusa development server
yarn partner-ui:dev         # Start partner UI dev server

# Build
yarn build                  # Build Medusa and resolve TypeScript aliases
yarn partner-ui:build       # Build partner UI preview

# Database
yarn predeploy              # Run migrations before deployment
yarn seed                   # Seed database with test data

# Generate UI
yarn generate-ui            # Generate admin UI components from schema
```

## Testing

```bash
# Run all test types
yarn test:integration:http              # HTTP integration tests
yarn test:integration:modules           # Module unit tests
yarn test:unit                          # Unit tests

# Run specific test files
TEST_TYPE=integration:http NODE_OPTIONS="--experimental-vm-modules" jest --testPathPattern="orders"
TEST_TYPE=integration:modules NODE_OPTIONS="--experimental-vm-modules" jest --testPathPattern="company"

# Run single test
TEST_TYPE=integration:http NODE_OPTIONS="--experimental-vm-modules" jest --testNamePattern="should create order"

# Run tests with verbose output
TEST_TYPE=integration:http NODE_OPTIONS="--experimental-vm-modules" jest --silent=false --runInBand

# Batched integration tests
yarn test:integration:http:batched
yarn test:integration:http:shared
```

## Code Style Guidelines

### TypeScript Configuration
- Target: ES2021
- Module: Node16 with Node16 resolution
- Strict null checks enabled
- Decorators enabled (experimentalDecorators, emitDecoratorMetadata)
- Paths: `@/*` maps to `./src/admin/*`

### Imports and Organization

**Import Order:**
1. External framework/library imports (@medusajs/*, @tanstack/*, ai, zod)
2. Internal module imports (`@/*`)
3. Relative imports (`./*`, `../*`)

**Example:**
```typescript
import { generateText } from "ai"
import { createOpenRouter } from "@openrouter/ai-sdk-provider"
import { z } from "zod"
import { MedusaService } from "@medusajs/framework/utils"
import Company from "./models/company"
```

### Naming Conventions

| Pattern | Convention | Example |
|---------|------------|---------|
| Files | kebab-case | `company-service.ts` |
| Classes | PascalCase | `CompanyService` |
| Variables/functions | camelCase | `createCompany()` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Private methods | prefix with `_` | `_internalMethod()` |
| Interfaces | PascalCase (no `I` prefix) | `CompanyConfig` |
| Types | PascalCase | `QueryStep` |
| Module constants | SCREAMING_SNAKE_CASE | `COMPANY_MODULE` |

### Error Handling

```typescript
// Use MedusaError from @medusajs/framework/utils
import { MedusaError } from "@medusajs/framework/utils"

throw new MedusaError(MedusaError.Types.NOT_FOUND, "Company not found")

// Error types: NOT_FOUND, INVALID_DATA, CONFLICT, UNAUTHORIZED, FORBIDDEN
```

### TypeScript Best Practices

- Use explicit types for function parameters and return types
- Use interfaces for object shapes, types for unions/primitives
- Prefer `zod` for runtime validation schemas
- Use `Record<K, V>` for map-like objects, not `{[key: string]: V}`
- Use `Partial<T>`, `Required<T>`, `Readonly<T>` for type modifiers

### Module Structure

```
src/modules/{module}/
├── index.ts              # Module definition with Module()
├── models/
│   └── {module}.ts       # DML model using model.define()
├── service.ts            # Service extending MedusaService()
├── migrations/           # MikroORM migrations
└── __tests__/            # Unit tests

src/api/
├── admin/                # Admin API routes
│   └── {resource}/
│       ├── route.ts      # Route handlers
│       └── validators.ts # Zod validation schemas
├── store/                # Storefront API routes
├── partners/             # Partner portal routes
└── middlewares.ts        # Global middleware definitions

src/workflows/
└── {workflow-name}/
    ├── index.ts          # Workflow definition
    └── steps/            # Step implementations

src/subscribers/
└── {event}.ts            # Event handlers
```

### Model Definition (DML)

```typescript
// src/modules/{module}/models/{module}.ts
import { model } from "@medusajs/framework/utils"

const Company = model.define("company", {
  id: model.id().primaryKey(),
  name: model.text(),
  email: model.text().optional(),
  created_at: model.dateTime().default("now"),
})

export default Company
```

### Service Definition

```typescript
// src/modules/{module}/service.ts
import { MedusaService } from "@medusajs/framework/utils"
import Company from "./models/company"

class CompanyService extends MedusaService({
  Company,
}) {}

export default CompanyService
```

### Module Definition

```typescript
// src/modules/{module}/index.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jaal-Yantra-Textiles/v2](https://github.com/Jaal-Yantra-Textiles/v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
