---
trigger: always_on
description: You are an expert AI assistant for the Nimara e-commerce project - a Next.js 16 full-stack platform with Saleor headless commerce backend.
---


# Nimara E-commerce - Cursor AI Rules

You are an expert AI assistant for the Nimara e-commerce project - a Next.js 16 full-stack platform with Saleor headless commerce backend.

## Core Tech Stack

- **Framework:** Next.js 16 (App Router, Server Components, Server Actions)
- **React:** 19
- **TypeScript:** Strict mode
- **Styling:** Tailwind CSS + Shadcn UI
- **GraphQL:** Saleor API with GraphQL Code Generator
- **Monorepo:** Turborepo + pnpm workspaces
- **Auth:** NextAuth.js v5
- **Payments:** Stripe
- **Testing:** Vitest + Playwright

## Project Structure

```
apps/
  ├── storefront/        # Main customer-facing app
  ├── stripe/            # Stripe integration app
  └── automated-tests/   # E2E tests
packages/
  ├── domain/            # Business logic (no external deps)
  ├── features/          # Feature implementations
  ├── infrastructure/    # External API integrations
  ├── foundation/        # Core utilities, hooks
  └── ui/                # Shared UI components
```

**Package dependencies:**

- `domain` & `foundation` are leaf packages
- `infrastructure` depends on `domain` + `foundation`
- `features` depends on all packages
- Apps can depend on any package

## Critical Rules

### 1. Server Components First (MOST IMPORTANT!)

- **Default:** ALL components are Server Components
- **Only use `'use client'` when you need:**
  - Event handlers (onClick, onChange, onSubmit)
  - React hooks (useState, useEffect, useContext, useReducer)
  - Browser APIs (window, localStorage, document)
  - Third-party client libraries
- **Benefits:** Better performance, SEO, smaller bundle
- **Server Components can:**
  - Be async functions
  - Fetch data directly
  - Access server-only resources

### 2. Type Safety

- TypeScript strict mode enabled everywhere
- **GraphQL types:** ALWAYS generated via codegen - NEVER write manually
- After schema changes: `pnpm run codegen`
- **Validation:** Use Zod schemas for user input
- **Public functions:** Must have explicit return types

### 3. File Naming

- **Components:** PascalCase (`ProductCard.tsx`)
- **Utilities:** camelCase (`formatPrice.ts`)
- **Types:** PascalCase + `.types.ts` (`Product.types.ts`)
- **GraphQL:** PascalCase + `.graphql` (`GetProducts.graphql`)
- **Tests:** Same name + `.test.ts`

### 4. Code Style

**Imports order:**

```typescript
import { Suspense } from "react"; // React/Next.js
import { useTranslations } from "next-intl"; // External
import { Product } from "@nimara/domain"; // Internal
import { formatPrice } from "./utils"; // Relative
import type { Props } from "./types"; // Types last
```

**Components:**

```typescript
// ✅ Preferred: Functional components with explicit types
export function ProductCard({ product }: { product: Product }) {
  return <div>{product.name}</div>;
}

// ❌ Avoid: Class components, default exports (except pages)
```

**Styling:**

```typescript
// ✅ Preferred: Tailwind utility classes
<div className="rounded-lg border p-4 shadow-sm hover:shadow-md">

// ❌ Avoid: Inline styles
```

## Common Patterns

### Data Fetching

**Server Components (default):**

```typescript
export default async function ProductsPage() {
  const products = await getProducts(); // Direct API call
  return <ProductList products={products} />;
}
```

**Server Actions (mutations):**

```typescript
"use server";

export async function addToCart(productId: string) {
  const session = await auth();
  if (!session) return { success: false, error: "Not authenticated" };

  const result = await saleorAPI.addToCart(productId);
  revalidatePath("/cart"); // ALWAYS revalidate after mutations

  return { success: true, data: result };
}
```

**GraphQL:**

```typescript
// 1. Define in .graphql file: infrastructure/product/queries/GetProducts.graphql
// 2. Run: pnpm run codegen
// 3. Use generated types:
import { GetProductsDocument } from "./queries.generated";

const { data } = await saleorClient.query({
  query: GetProductsDocument,
  variables: { first: 10 },
});
```

### State Management

- **Server state:** React Server Components (default)
- **Client state:** `useState`, `useReducer`
- **URL state:** `searchParams`
- **Form state:** Server Actions + `useFormState`

### Error Handling

```typescript
"use server";

export async function updateProfile(data: FormData) {
  try {
    const validated = profileSchema.parse(data); // Zod validation
    await updateUser(validated);
    revalidatePath("/account");
    return { success: true };
  } catch (error) {
    console.error("Profile update failed:", error);
    return { success: false, error: "Failed to update profile" };
  }
}
```

## When Adding New Features

1. **Check existing patterns** - Look for similar features first
2. **Choose correct package:**
   - `domain` - Pure business logic, entities
   - `features` - Feature implementations with components
   - `infrastructure` - External API integrations
   - `foundation` - Utilities, helpers, hooks
3. **Add types** in `@nimara/domain` if shared
4. **Create GraphQL operations:**
   - Add `.graphql` file in `infrastructure`
   - Run `pnpm run codegen`
5. **Add translations** - All user-facing text uses `next-intl`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirumee/nimara-ecommerce](https://github.com/mirumee/nimara-ecommerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
