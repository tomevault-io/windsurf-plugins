---
trigger: always_on
description: Technical reference for Claude Code (and other agents) when working with Next Stripe Store.
---

# CLAUDE.md

Technical reference for Claude Code (and other agents) when working with Next Stripe Store.

## Quick Reference

| Command | Purpose |
|---------|---------|
| `bun dev` | Start development server |
| `bun run lint` | Check code with Biome |
| `bun run format` | Format code with Biome |
| `bunx tsc --noEmit` | Type check without emit |
| `bunx drizzle-kit migrate` | Apply database migrations |

## Project Overview

**Next Stripe Store** is a Next.js 16.1 + React 19 e-commerce app with:

- **Next.js 16.1** – App Router, RSC, React Compiler (`reactCompiler: true`), `cacheComponents: true`
- **React 19** – RSC + streaming
- **Catalog in Postgres** – Products/variants/collections stored in DB via Drizzle ORM
- **Stripe** – Checkout Sessions + Webhooks, plus a sync layer that pushes DB catalog to Stripe Products/Prices
- **Better Auth** – Email/password auth backed by Drizzle adapter, with DB-backed roles (`USER`/`ADMIN`)
- **Zustand** – Client cart state (optimistic UI) hydrated from server
- **Tailwind CSS v4** – Configured via PostCSS (no `tailwind.config.ts`)
- **Biome** – Lint/format (no ESLint/Prettier)
- **TypeScript** – Strict type checking

## Architecture

### Catalog + Commerce Layer (`src/lib/commerce.ts`)

This project’s **source of truth is the database**. `commerce.*` queries the DB and exposes a UI-friendly shape:

- `ProductVariant.id` is the **Stripe Price ID** (`stripePriceId`) for cart/checkout compatibility.

```typescript
// Product listing
const { data: products } = await commerce.productBrowse({ limit: 12 });

// Single product by slug or ID
const product = await commerce.productGet({ idOrSlug: "my-product" });

// Collections (DB-backed)
const { data: collections } = await commerce.collectionBrowse({ limit: 5 });

// Variant with product (used by cart hydration)
const result = await commerce.getVariantWithProduct(priceId);
```

### Product Sync to Stripe (`src/lib/product-sync.ts`)

Admin workflows push DB products/variants to Stripe so Checkout can price items by Stripe Price ID.

- Product → Stripe Product (`products.stripeProductId`)
- Variant → Stripe Price (`product_variants.stripePriceId`)
- Variant options are written to Stripe Price `metadata` with keys starting with `option...`

### Cart System (`src/app/cart/actions.ts`)

- Cart stored in httpOnly `cart` cookie (Stripe Price IDs + quantities)
- Single currency per cart enforced
- Server actions: `getCart`, `addToCart`, `removeFromCart`, `setCartQuantity`, `startCheckout`

```typescript
// Add item to cart
const result = await addToCart(variantId, quantity);

// Start Stripe Checkout
const { url } = await startCheckout();
```

### Orders + Webhooks

- `POST /api/webhooks/stripe` verifies signature with `STRIPE_WEBHOOK_SECRET`
- Creates and updates orders via `src/lib/orders.ts`
- Handles:
  - `checkout.session.completed` → create order + items
  - `payment_intent.succeeded` → mark paid
  - `charge.refunded` → mark refunded / partially_refunded

### Data Model (High Level)

```
DB Product        → products (id, slug, stripeProductId?, syncStatus, ...)
DB Variant        → product_variants (id, stripePriceId?, price, currency, options, syncStatus, ...)
Cart line item    → Stripe Price ID + quantity (stored in httpOnly cookie)
Order persistence → orders + order_items (created from Stripe sessions via webhook)
```

## File Structure

```
src/
├── app/
│   ├── (store)/                     # Store routes
│   ├── (auth)/                      # Login/register
│   ├── admin/                       # Admin pages
│   ├── api/
│   │   ├── auth/[...betterAuth]/    # Better Auth handler
│   │   ├── admin/*                  # Admin endpoints (CRUD + sync)
│   │   └── webhooks/stripe/         # Stripe webhook handler
│   └── cart/                        # Server cart actions + cart sidebar UI
├── components/                      # UI + domain components
├── db/                              # Drizzle schema + db client
├── lib/                             # commerce, orders, auth, product sync, money, utils
└── utils/                           # shared pure helpers (e.g. cart totals)
```

## Code Patterns

### Server Components with Caching

```tsx
async function ProductList() {
  "use cache";
  cacheLife("seconds");
  
  const { data: products } = await commerce.productBrowse({ limit: 12 });
  return <ProductGrid products={products} />;
}

// Always wrap in Suspense
<Suspense fallback={<Skeleton />}>
  <ProductList />
</Suspense>
```

### Client Components with Cart

```tsx
"use client";

import { addToCart } from "@/app/cart/actions";
import { useCart } from "@/components/cart/use-cart";

function AddButton({ variantId }: { variantId: string }) {
  const openCart = useCart((state) => state.openCart);
  const add = useCart((state) => state.add);
  const sync = useCart((state) => state.sync);
  
  const handleAdd = async () => {
    openCart();
    add({
      quantity: 1,
      productVariant: {
        id: variantId,
        // price/currency/images/combinations/product omitted for brevity
      },
    });
    const result = await addToCart(variantId, 1);
    if (result?.cart) sync(result.cart);
  };
  
  return <button onClick={handleAdd}>Add to Cart</button>;
}
```

### Price Formatting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guillermolg00/nextjs-stripe-store](https://github.com/guillermolg00/nextjs-stripe-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
