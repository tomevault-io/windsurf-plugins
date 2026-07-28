---
trigger: always_on
description: Transaction domain vertical slice architecture guidelines for adding new features
---


# Transaction Domain - Vertical Slice Architecture

The transaction domain uses a **vertical slice architecture** where each feature contains its complete stack:
- **API**: tRPC procedures
- **Backend**: Lambda functions (AWS Lambda handlers)
- **Infra**: Step Functions state machines
- **Stack**: Infrastructure exports
- **Web**: React components, pages, and routes

## Adding a New Feature

When asked to add a new feature, follow these steps in order:

### 1. Create Feature Folder Structure

Create the following directory structure:
```
features/{feature-name}/
├── api/
├── backend/
├── infra/
├── stack/
└── web/
    ├── components/
    └── pages/
```

**Use kebab-case for feature names** (e.g., `transfer-funds`, `transaction-history`)

### 2. Create API Procedure

File: `features/{feature-name}/api/{featureName}Procedures.ts`

**Template:**
```typescript
import { publicProcedure } from '@purple-stack/trpc-api/trpc'
import { z } from 'zod'

// For queries:
export const {featureName} = publicProcedure
  .input(z.object({
    // Define input schema
  }))
  .query(async ({ input }) => {
    // Implementation
    return { /* result */ }
  })

// For mutations:
export const {featureName} = publicProcedure
  .input(z.object({
    // Define input schema
  }))
  .mutation(async ({ input }) => {
    // Implementation
    return { /* result */ }
  })
```

**Naming conventions:**
- File: `{featureName}Procedures.ts` (camelCase)
- Export: `{featureName}` (camelCase, matches feature folder name in kebab-case converted)

### 3. Register API Procedure in Domain Router

File: `api/transactionRouter.ts`

**Add import:**
```typescript
import { {featureName} } from '../features/{feature-name}/api/{featureName}Procedures'
```

**Add to router:**
```typescript
export const transactionRouter = router({
  // ... existing procedures
  {featureName}
})
```

### 4. Create Web Components

File: `features/{feature-name}/web/components/{ComponentName}.tsx`

**Template:**
```typescript
import { Button } from '../../../../web/components'
import '../../../../web/styles/shared.css'
import './{ComponentName}.css'

export interface {ComponentName}Props {
  // Define props
}

export function {ComponentName}({ /* props */ }: {ComponentName}Props) {
  return (
    <div className="{component-name} transaction-card">
      {/* Component content */}
    </div>
  )
}
```

**Requirements:**
- Always import shared styles: `'../../../../web/styles/shared.css'`
- Import component-specific CSS: `'./{ComponentName}.css'`
- Use shared components from `'../../../../web/components'`
- Use shared CSS classes like `transaction-card`, `transaction-page`, `transaction-input`
- Export TypeScript types for props

**Create co-located CSS file:** `features/{feature-name}/web/components/{ComponentName}.css`

### 5. Create Page Component

File: `features/{feature-name}/web/pages/{FeatureName}Page.tsx`

**Template:**
```typescript
import { appClient } from '@purple-stack/trpc-api/trpcClient'
import '../../../../web/styles/shared.css'
import './{FeatureName}Page.css'
import { ComponentName } from '../components/{ComponentName}'

export function {FeatureName}Page() {
  // Use appClient.transaction.{featureName} to call API
  return (
    <main className="{feature-name}-page transaction-page">
      <header className="{feature-name}-page__header">
        <h1>{Feature Name}</h1>
      </header>
      <section className="{feature-name}-page__content">
        <ComponentName />
      </section>
    </main>
  )
}
```

**Requirements:**
- Always import shared styles first
- Use `transaction-page` class for main layout
- Use BEM naming for feature-specific classes: `{feature-name}-page__header`
- Import and use feature components

**Create co-located CSS file:** `features/{feature-name}/web/pages/{FeatureName}Page.css`

### 6. Create Route Factory

File: `features/{feature-name}/web/{feature-name}.route.tsx`

**Template:**
```typescript
import type { AnyRootRoute } from '@tanstack/react-router'
import { createRoute } from '@tanstack/react-router'
import { {FeatureName}Page } from './pages/{FeatureName}Page'

export const create{FeatureName}Route = <TRootRoute extends AnyRootRoute>(
  rootRoute: TRootRoute
) =>
  createRoute({
    getParentRoute: () => rootRoute,
    path: '{feature-name}',
    component: {FeatureName}Page
  })
```

**Requirements:**
- Function name: `create{FeatureName}Route` (PascalCase)
- Path: `'{feature-name}'` (kebab-case, matches folder name)
- Use factory pattern with generic `TRootRoute extends AnyRootRoute`

### 7. Export Public API

File: `features/{feature-name}/web/index.ts`

**Template:**
```typescript
// Export component types
export type { {ComponentName}Props } from './components/{ComponentName}'

// Export components
export { {ComponentName} } from './components/{ComponentName}'

// Export page
export { {FeatureName}Page } from './pages/{FeatureName}Page'

// Export route factory
export { create{FeatureName}Route } from './{feature-name}.route'
```

**Requirements:**
- Export all public components and their types
- Export the page component
- Export the route factory function

### 8. Register Route in Domain Routes

File: `web/routes/index.ts`

**Add export:**
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purple-technology/purple-stack](https://github.com/purple-technology/purple-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
