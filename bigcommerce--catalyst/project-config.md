---
trigger: always_on
description: This document provides guidance for Large Language Models (LLMs) working with the BigCommerce Catalyst codebase, focusing on the **Next.js App Router application** architecture, data fetching patterns, and key design principles.
---

# AGENTS.md

## BigCommerce Catalyst Codebase Overview

This document provides guidance for Large Language Models (LLMs) working with the BigCommerce Catalyst codebase, focusing on the **Next.js App Router application** architecture, data fetching patterns, and key design principles.

**Catalyst is built as a Next.js App Router application** with React Server Components, enabling server-side data fetching, automatic code splitting, and optimal performance for e-commerce workloads.

## Repository Structure

The main Next.js application is located in the `/core` directory, which contains the complete e-commerce storefront implementation. Other packages exist outside of `/core` but are not the primary focus for most development work.

## Proxy Architecture

The application uses the Next.js 16 proxy pattern (`proxy.ts`) with a composed proxy stack that significantly alters the default Next.js routing behavior. The proxy composition (in the `proxies/` directory) includes authentication, internationalization, analytics, channel handling, and most importantly, custom routing.

### Custom Routing with `with-routes`

The `with-routes` proxy is the most critical component that overrides Next.js's default path-based routing. Instead of relying on file-based routing, this proxy:

1. **Queries the BigCommerce GraphQL API** to resolve incoming URL paths to specific entity types (products, categories, brands, blog posts, pages).

2. **Rewrites requests** to internal Next.js routes based on the resolved entity type.

3. **Handles redirects** automatically based on BigCommerce's redirect configuration.

This means that URLs like `/my-product-name` can resolve to `/en/product/123` internally, providing flexible URL structure while maintaining SEO-friendly paths.

## Data Fetching and Partial Prerendering (PPR)

### PPR Configuration

The application uses Next.js Partial Prerendering with incremental adoption. This allows static parts of pages to be prerendered while dynamic content streams in.

### Streamable Pattern

The `Streamable<T>` pattern is a core architectural concept that enables efficient data streaming and React Server Component compatibility.

#### What is Streamable?

```typescript
export type Streamable<T> = T | Promise<T>;
```

A `Streamable<T>` represents data that can be either:
- **Immediate**: Already resolved data of type `T`
- **Deferred**: A Promise that will resolve to type `T`

#### Core Streamable API

Located in `core/vibes/soul/lib/streamable.tsx`, the Streamable system provides:

**`Streamable.from()`** - Creates a streamable from a lazy promise factory:
```typescript
const streamableProducts = Streamable.from(async () => {
  const customerToken = await getSessionCustomerAccessToken();
  const currencyCode = await getPreferredCurrencyCode();
  return getProducts(customerToken, currencyCode);
});
```

**`Streamable.all()`** - Combines multiple streamables with automatic caching:
```typescript
const combined = Streamable.all([
  streamableProducts,
  streamableCategories,
  streamableUser
]);
```

**`useStreamable()`** - Hook for consuming streamables in components:
```typescript
function MyComponent({ data }: { data: Streamable<Product[]> }) {
  const products = useStreamable(data);
  return <div>{products.map(...)}</div>;
}
```

**`<Stream>` Component** - Provides Suspense boundary for streamable data:
```tsx
<Stream value={streamableProducts} fallback={<ProductSkeleton />}>
  {(products) => <ProductList products={products} />}
</Stream>
```

#### Streamable Benefits

- **Performance**: Enables concurrent data fetching and streaming
- **Caching**: Automatic promise deduplication and stability
- **Flexibility**: Works with both sync and async data
- **Suspense Integration**: Built-in React Suspense support
- **Composition**: Easy chaining and combination of data sources

### Data Fetching Best Practices

1. **Use React's `cache()` function** for server-side data fetching to memoize function results and prevent repeated fetches or computations **per request** (React will invalidate the cache for all memoized functions for each server request).

2. **Implement proper cache strategies** based on whether user authentication is present.

3. **Leverage Streamable for progressive enhancement** where static content loads immediately and dynamic content streams in.

## GraphQL API Client

### Centralized Client Configuration

All interactions with the BigCommerce Storefront GraphQL API should use the centralized GraphQL client. This client provides:

- Automatic channel ID resolution based on locale
- Proper authentication token handling
- Request/response logging in development
- Error handling with automatic auth redirects
- IP address forwarding for personalization

### Usage Pattern

Always import and use the configured client rather than making direct API calls. The client handles all the necessary headers, authentication, and channel context automatically.

## UI Design System (Vibes)

### Architecture Overview


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigcommerce/catalyst](https://github.com/bigcommerce/catalyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
