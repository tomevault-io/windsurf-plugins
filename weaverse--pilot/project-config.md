---
trigger: always_on
description: This file provides guidance to AI agents (Claude, GitHub Copilot, Cursor, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (Claude, GitHub Copilot, Cursor, etc.) when working with code in this repository.

## Project Overview

This is **Pilot**, a Shopify Hydrogen theme powered by Weaverse - a visual page builder for Hydrogen storefronts. The project is built with React 19, TypeScript, React Router 7, and Tailwind CSS v4. It runs on Node.js 20+ and uses Biome for linting/formatting.

## Architecture Overview

### Route Structure
Routes are defined in `app/routes.ts` using React Router v7's programmatic routing with Hydrogen's `hydrogenRoutes` wrapper (migrated from flat file-based routing in v7.0.0). All routes support internationalization via the `:locale?` prefix:
- Homepage: `routes/home.tsx`
- Products: `routes/products/product.tsx` (`:productHandle`)
- Collections: `routes/collections/collection.tsx` (`:collectionHandle`)
- Account: `routes/account/*` (layout-based with nested routes)
- API routes: `routes/api/*`
- Cart operations: `routes/cart/*`
- Policies & Pages: `routes/pages/regular-page.tsx`, `routes/policies/policy.tsx`
- Blogs: `routes/blogs/blog.tsx`, `routes/blogs/article.tsx`
- SEO: `routes/seo/robots.ts`, `routes/seo/sitemap.ts`

**Adding New Routes**: Always add new routes to `app/routes.ts` using the `route()`, `index()`, `layout()`, and `prefix()` helpers from `@react-router/dev/routes`.

### Key Architectural Patterns

1. **Parallel Data Loading**: Every page route loads Weaverse data alongside GraphQL queries using `Promise.all()`:
   ```typescript
   const [{ shop, product }, weaverseData, productReviews] = await Promise.all([
     storefront.query(PRODUCT_QUERY, { variables }),
     weaverse.loadPage({ type: "PRODUCT", handle }),
     getJudgeMeProductReviews({ context, handle }),
   ]);
   ```

2. **Component Structure**:
   - `/app/sections/` - Weaverse visual builder sections with schema exports and optional loaders
   - `/app/components/` - Reusable UI components organized by feature (cart/, product/, layout/, customer/)
   - Each Weaverse section must export: default component + schema + optional loader

3. **Data Fetching**:
   - GraphQL fragments in `/app/graphql/fragments.ts`
   - Complete queries in `/app/graphql/queries.ts`
   - Route loaders handle all data fetching server-side
   - Use `routeHeaders` for consistent cache control

4. **Styling**:
   - Tailwind CSS v4 with custom utilities
   - class-variance-authority (cva) for component variants
   - Use the `cn()` utility from `/app/utils/cn.ts` for class merging
   - Biome's `useSortedClasses` enabled for `clsx`, `cva`, and `cn` functions

5. **Type Safety**:
   - GraphQL types are auto-generated via codegen
   - Path alias `~/` maps to `/app/` directory
   - Strict TypeScript configuration
   - Two separate codegen outputs:
     - `storefront-api.generated.d.ts` - For all storefront queries (excludes account routes)
     - `customer-account-api.generated.d.ts` - For customer account queries (only in `*.account*.{ts,tsx,js,jsx}` files)

### Important Integrations

- **Weaverse**: Visual page builder - sections must be registered in `/app/weaverse/components.ts`
- **Judge.me**: Product reviews integration via utilities in `/app/utils/judgeme.ts`
- **Combined Listings**: Product grouping system via `/app/utils/combined-listings.ts`
- **Customer Accounts**: New Shopify Customer Account API support (OAuth-based)
- **Radix UI**: For accessible UI primitives (accordion, dialog, dropdown, etc.)
- **Swiper**: For carousel/slideshow functionality
- **Zustand**: For lightweight state management

### Weaverse Section Development

1. **Creating a New Section**:
  ```typescript
  // app/sections/my-section/index.tsx
  import { createSchema, type HydrogenComponentProps } from '@weaverse/hydrogen';
  import type { ComponentRef } from 'react';

  interface MyProps extends HydrogenComponentProps {
    heading: string;
    ref?: React.Ref<HTMLDivElement>;
  }

  const MySection = ({ ref, ...props }: MyProps) => {
    // Component implementation
  };

  export default MySection;

  export const schema = createSchema({
    type: 'my-section',
    title: 'My Section',
    settings: [/* ... */]
  });

  // Optional loader for server-side data fetching
  export const loader = async ({ weaverse, data }) => {
    // Fetch data
  };
  ```

2. **Register in `/app/weaverse/components.ts`**:
   ```typescript
   import * as MySection from "~/sections/my-section";
   export const components = [
     // ... existing components
     MySection,
   ];
   ```

### Customer Account Local Development

See [`.weaverse/docs/customer-account-local-dev.md`](.weaverse/docs/customer-account-local-dev.md) for the full guide on running locally with Customer Account API support (`nr dev:ca`).

### Environment Configuration

Required environment variables are defined in `env.d.ts`. The project uses `@shopify/hydrogen` and `@shopify/remix-oxygen` for environment handling.

### Biome Configuration

The project extends from `ultracite` and `@weaverse/biome` configurations with these customizations:
- Double quotes for strings
- Semicolons always
- Trailing commas
- Max cognitive complexity: 50
- Sorted Tailwind classes in `clsx`, `cva`, and `cn` functions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Weaverse/pilot](https://github.com/Weaverse/pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
