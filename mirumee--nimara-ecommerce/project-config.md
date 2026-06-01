---
trigger: always_on
description: This file provides comprehensive context for AI coding assistants (GitHub Copilot, Cursor, etc.) working on the Nimara e-commerce project. It enables AI to understand the project structure, conventions, and patterns to generate accurate, contextual code.
---

# Nimara E-commerce - AI Assistant Context

This file provides comprehensive context for AI coding assistants (GitHub Copilot, Cursor, etc.) working on the Nimara e-commerce project. It enables AI to understand the project structure, conventions, and patterns to generate accurate, contextual code.

---

## Project Type

Full-stack e-commerce platform using Next.js 16 with Saleor headless commerce backend. Built with a monorepo architecture for multi-region, global brands.

---

## Tech Stack

### Frontend

- **Framework:** Next.js 16 (App Router, Server Components, Server Actions)
- **React:** Version 19
- **TypeScript:** Strict mode enabled
- **Styling:** Tailwind CSS 3.x with utility-first approach
- **UI Components:** Shadcn UI (built on Radix UI primitives)
- **Forms:** React Hook Form with Zod validation

### Backend & APIs

- **Commerce Backend:** Saleor GraphQL API
- **Payment Processing:** Stripe Payment Element
- **Authentication:** NextAuth.js v5 (Auth.js)
- **CMS:** Saleor (default) or ButterCMS (optional)
- **Search:** Saleor (default) or Algolia (optional)

### Monorepo & Tools

- **Monorepo:** Turborepo with pnpm workspaces
- **Package Manager:** pnpm 9.x
- **Code Generation:** GraphQL Code Generator
- **Testing:** Vitest (unit), Playwright (E2E)
- **Linting:** ESLint with custom configs
- **Formatting:** Prettier with Tailwind plugin

### Deployment

- **Hosting:** Vercel (multi-environment setup)
- **Error Tracking:** Sentry
- **Logging:** Pino structured logging

---

## Project Structure

```
nimara-ecommerce/
├── apps/
│   ├── storefront/        # Main customer-facing Next.js app
│   ├── stripe/            # Stripe integration Next.js app
│   ├── automated-tests/   # Playwright E2E test suite
│   └── docs/              # Documentation site (Nextra)
├── packages/
│   ├── domain/            # Business logic and entities (no external deps)
│   ├── features/          # Feature implementations (cart, checkout, products)
│   ├── infrastructure/    # External API integrations (Saleor, Stripe)
│   ├── foundation/        # Core utilities, hooks, helpers
│   ├── ui/                # Shared UI components (Shadcn-based)
│   ├── translations/      # i18n message catalogs
│   ├── codegen/           # GraphQL code generation config
│   ├── config/            # Shared configurations
│   └── tsconfig/          # TypeScript configurations
└── terraform/             # Infrastructure as Code
```

**Package Dependencies:**

- `domain` & `foundation` are leaf packages (no internal dependencies)
- `infrastructure` depends on `domain` and `foundation`
- `features` depends on `domain`, `infrastructure`, `foundation`, and `ui`
- Apps can depend on any package

---

## Key Principles

### 1. Server Components First

- **Default:** All components are Server Components
- **Rule:** Only add `'use client'` when you need:
  - Interactivity (onClick, onChange, event handlers)
  - React hooks (useState, useEffect, useContext)
  - Browser APIs (localStorage, window, document)
  - Third-party libraries that use browser features
- **Benefits:** Better performance, SEO, smaller bundle size

### 2. Type Safety First

- **TypeScript strict mode** is enabled across all packages
- **GraphQL types** are generated via codegen - never write them manually
- **Validation** uses Zod schemas for runtime type checking
- **Public functions** must have explicit return types

### 3. Composition Over Configuration

- Build complex UIs from simple, focused components
- Prefer component composition over prop drilling
- Server Components wrap Client Components for optimal performance
- Keep components under 200 lines

### 4. Performance Matters

- Leverage Next.js caching (Router Cache, Full Route Cache, Data Cache)
- Use ISR (Incremental Static Regeneration) for frequently updated content
- Implement streaming with Suspense boundaries
- Optimize images with Next/Image component
- Code-split heavy components with dynamic imports

### 5. Internationalization Ready

- All user-facing text must use translation keys
- Use `next-intl` for internationalization
- Support multiple currencies and locales
- Format dates, numbers, and prices according to locale

---

## Common Patterns

### Data Fetching

**Server Components (default):**

```typescript
// app/products/page.tsx
export default async function ProductsPage() {
  // Direct API call in Server Component
  const products = await getProducts();

  return <ProductList products={products} />;
}
```

**Server Actions (mutations):**

```typescript
// actions/cart.ts
"use server";

export async function addToCart(productId: string) {
  const session = await auth();
  if (!session) {
    return { success: false, error: "Not authenticated" };
  }

  const result = await saleorAPI.addToCart(productId);
  revalidatePath("/cart");

  return { success: true, data: result };
}
```

**GraphQL Queries:**

```typescript
// 1. Define query in .graphql file
// infrastructure/product/queries/GetProducts.graphql

// 2. Run codegen
// pnpm run codegen

// 3. Use generated types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirumee/nimara-ecommerce](https://github.com/mirumee/nimara-ecommerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
