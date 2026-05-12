---
trigger: always_on
description: This is a headless e-commerce storefront built with Next.js 16 and React 19, using the Spree Commerce API v3 via `@spree/sdk`.
---

# Claude Code Rules for Next.js Spree Storefront

## Project Overview

This is a headless e-commerce storefront built with Next.js 16 and React 19, using the Spree Commerce API v3 via `@spree/sdk`.

## Tech Stack

- **Framework:** Next.js 16 (App Router)
- **React:** 19 (with new features like `use()`, Actions, improved Suspense)
- **Styling:** Tailwind CSS
- **API Client:** `@spree/sdk`
- **Language:** TypeScript

## Project Structure

```
src/
├── app/                          # Next.js App Router
│   └── [country]/[locale]/       # Internationalized routes
│       ├── (checkout)/           # Checkout route group (minimal layout)
│       └── (storefront)/         # Storefront route group (full layout)
├── components/                   # Reusable UI components
├── contexts/                     # React Context providers
├── lib/
│   └── data/                     # Server Actions for data fetching
└── types/                        # TypeScript type definitions
```

## React 19 Best Practices

### Avoid Unnecessary useEffect

React 19 provides better patterns for many cases where `useEffect` was previously required. Follow https://react.dev/learn/you-might-not-need-an-effect

#### Don't Use useEffect For:

**1. Transforming data for rendering**
```typescript
// ❌ Bad - useEffect for derived state
const [fullName, setFullName] = useState("");
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// ✅ Good - compute during render
const fullName = `${firstName} ${lastName}`;

// ✅ Good - useMemo for expensive calculations
const sortedProducts = useMemo(
  () => products.sort((a, b) => a.price - b.price),
  [products]
);
```

**2. Resetting state when props change**
```typescript
// ❌ Bad - useEffect to reset state
useEffect(() => {
  setSelectedVariant(null);
}, [productId]);

// ✅ Good - use key to reset component state
<ProductDetails key={productId} product={product} />

// ✅ Good - compute initial state from props
const [selectedVariant, setSelectedVariant] = useState(() => {
  return product.default_variant || product.variants[0];
});
```

**3. Fetching data in response to user events**
```typescript
// ❌ Bad - useEffect triggered by state
const [query, setQuery] = useState("");
useEffect(() => {
  fetchResults(query);
}, [query]);

// ✅ Good - fetch in event handler
const handleSearch = async (searchQuery: string) => {
  setQuery(searchQuery);
  const results = await fetchResults(searchQuery);
  setResults(results);
};

// ✅ Better - use Server Actions
const [results, searchAction] = useActionState(searchProducts, []);
```

**4. Initializing the application**
```typescript
// ❌ Bad - useEffect for one-time init
useEffect(() => {
  loadAnalytics();
}, []);

// ✅ Good - module-level initialization
if (typeof window !== "undefined") {
  loadAnalytics();
}

// ✅ Good - check if already initialized
let didInit = false;
function App() {
  if (!didInit) {
    didInit = true;
    loadAnalytics();
  }
  return null;
}
```

#### When useEffect IS Appropriate:

- **Synchronizing with external systems** (DOM APIs, third-party widgets, network)
- **Setting up subscriptions** (WebSocket, event listeners)
- **Browser-only effects** (focus management, scroll position)
- **Intersection Observer, Resize Observer**

### Use Server Components by Default

```typescript
// ✅ Good - Server Component (default, no "use client")
// src/app/[country]/[locale]/(storefront)/products/page.tsx
import { getProducts } from "@/lib/data/products";

export default async function ProductsPage() {
  const products = await getProducts();
  return <ProductList products={products} />;
}
```

Only add `"use client"` when you need:
- Event handlers (onClick, onChange, etc.)
- useState, useReducer, useEffect, useContext
- Browser-only APIs
- Custom hooks that use state/effects

### Use Server Actions for Mutations

```typescript
// src/lib/data/cart.ts
"use server";

export async function addToCart(variantId: string, quantity: number) {
  const cart = await getOrCreateCart();
  const client = await getSpreeClient();

  return client.orders.lineItems.create(
    cart.id,
    { variant_id: variantId, quantity },
    { orderToken: cart.token }
  );
}

// Component usage
import { addToCart } from "@/lib/data/cart";

function AddToCartButton({ variantId }: { variantId: string }) {
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(async () => {
      await addToCart(variantId, 1);
    });
  };

  return (
    <button onClick={handleClick} disabled={isPending}>
      {isPending ? "Adding..." : "Add to Cart"}
    </button>
  );
}
```

### Use React 19 `use()` for Promises

```typescript
// ✅ Good - use() with Suspense
import { use, Suspense } from "react";

interface ProductDetailsProps {
  productPromise: Promise<Product>;
}

function ProductDetails({ productPromise }: ProductDetailsProps) {
  const product = use(productPromise);
  return <div>{product.name}</div>;
}

// Parent component
function ProductPage({ id }: { id: string }) {
  const productPromise = getProduct(id); // Don't await here

  return (
    <Suspense fallback={<ProductSkeleton />}>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spree/storefront](https://github.com/spree/storefront) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
