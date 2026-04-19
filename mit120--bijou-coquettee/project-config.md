---
trigger: always_on
description: This is a **Medusa v2** e-commerce platform consisting of:
---

# Bijou Coquettee - Medusa.js E-commerce Platform
# Cursor AI Rules & Development Guidelines

## 🏗️ Project Structure Overview

This is a **Medusa v2** e-commerce platform consisting of:
1. **Backend**: `bijou-coquettee/` - Medusa.js application server
2. **Storefront**: `bijou-coquettee-storefront/` - Next.js 14+ App Router storefront

---

## 📚 Primary Documentation Reference

**Official Docs**: https://docs.medusajs.com/
**Version**: v2.11.1

### Key Documentation Sections:
- **Framework Guide**: https://docs.medusajs.com/learn
- **API Routes**: https://docs.medusajs.com/learn/basics/api-routes
- **Workflows**: https://docs.medusajs.com/learn/basics/workflows
- **Data Models**: https://docs.medusajs.com/learn/basics/data-models
- **Modules**: https://docs.medusajs.com/learn/basics/modules
- **Admin Customization**: https://docs.medusajs.com/learn/customization/customize-admin
- **Commerce Modules**: https://docs.medusajs.com/resources/references
- **Storefront Development**: https://docs.medusajs.com/resources/storefront-development

---

## 🎯 Backend Development Rules (bijou-coquettee/)

### 1. API Routes (`src/api/`)

**Structure**:
- `src/api/admin/` - Admin API endpoints (requires authentication)
- `src/api/store/` - Storefront API endpoints (public/customer)

**Route Creation Pattern**:
```typescript
// src/api/[admin|store]/[route-name]/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  // Use dependency injection
  const query = req.scope.resolve("query")
  
  // Your logic here
  
  res.json({ data: result })
}
```

**Rules**:
- Always use `MedusaRequest` and `MedusaResponse` types
- Use dependency injection via `req.scope.resolve()`
- Follow REST conventions (GET, POST, PUT, DELETE, PATCH)
- Admin routes should validate admin authentication
- Store routes should handle guest and authenticated customers

**Documentation**: https://docs.medusajs.com/learn/basics/api-routes

---

### 2. Workflows (`src/workflows/`)

**Purpose**: Orchestrate complex business logic with rollback support

**Workflow Creation Pattern**:
```typescript
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"

export const myCustomWorkflow = createWorkflow(
  "my-custom-workflow",
  function (input: InputType) {
    // Define steps
    const step1Result = step1(input)
    const step2Result = step2(step1Result)
    
    return new WorkflowResponse({
      result: step2Result
    })
  }
)
```

**Rules**:
- Workflows should be composable and reusable
- Each step should have compensation (rollback) logic
- Use built-in workflows when available
- Name workflows descriptively: `[entity]-[action]-workflow`
- Store workflows in `src/workflows/[feature]/`

**Documentation**: 
- https://docs.medusajs.com/learn/basics/workflows
- https://docs.medusajs.com/resources/references/workflows-sdk

---

### 3. Data Models (`src/modules/`)

**Data Model Definition**:
```typescript
import { model } from "@medusajs/framework/utils"

export const MyModel = model.define("my_model", {
  id: model.id().primaryKey(),
  name: model.text(),
  description: model.text().nullable(),
  // ... other fields
})
```

**Rules**:
- Use Data Model Language (DML) for defining models
- Follow snake_case for database column names
- Use camelCase for TypeScript property names
- Always define proper relationships (hasOne, hasMany, belongsTo)
- Add indexes for frequently queried fields
- Use enums for status fields

**Documentation**: 
- https://docs.medusajs.com/learn/basics/data-models
- https://docs.medusajs.com/resources/references/data-model

---

### 4. Modules (`src/modules/`)

**Module Structure**:
```
src/modules/[module-name]/
├── models/
│   └── [model-name].ts
├── services/
│   └── [service-name].ts
├── workflows/
│   └── [workflow-name].ts
├── migrations/
│   └── [timestamp]-[description].ts
└── index.ts
```

**Service Pattern**:
```typescript
import { MedusaService } from "@medusajs/framework/utils"

class MyModuleService extends MedusaService({
  MyModel,
}) {
  async customMethod(id: string) {
    // Your custom logic
  }
}

export default MyModuleService
```

**Rules**:
- One module per business domain
- Export module definition in `index.ts`
- Services should extend `MedusaService` for CRUD operations
- Use transactions for multi-step operations
- Module names should be lowercase-hyphenated

**Documentation**: https://docs.medusajs.com/learn/basics/modules

---

### 5. Links (`src/links/`)

**Purpose**: Define relationships between modules

**Link Definition**:
```typescript
import { defineLink } from "@medusajs/framework/utils"
import MyModule from "../modules/my-module"
import ProductModule from "@medusajs/framework/product"

export default defineLink(
  MyModule.linkable.myModel,
  ProductModule.linkable.product
)
```

**Rules**:
- Links enable cross-module queries
- Use for relationships between different modules
- Define in `src/links/` directory
- Always specify both sides of the relationship

**Documentation**: https://docs.medusajs.com/learn/basics/modules/module-links

---

### 6. Subscribers (`src/subscribers/`)

**Purpose**: React to domain events

**Subscriber Pattern**:
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MIT120) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
