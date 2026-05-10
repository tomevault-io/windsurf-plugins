---
trigger: always_on
description: This document provides detailed instructions for AI assistants working on this headless WordPress e-commerce frontend.
---

# AGENTS.md - AI Assistant Guidelines

This document provides detailed instructions for AI assistants working on this headless WordPress e-commerce frontend.

## Project Context

This is a **production-ready headless e-commerce storefront** that:
- Fetches content from WordPress via GraphQL (WPGraphQL plugin)
- Manages products/orders via WooCommerce REST API v3
- Uses JWT for customer authentication
- Persists cart and auth state to localStorage

## Before Making Changes

### 1. Understand the Architecture
- **Server Components** render on the server (default in App Router)
- **Client Components** require `'use client'` directive
- API routes in `src/app/api/` handle sensitive operations
- State lives in Zustand stores, not React context

### 2. Check Existing Patterns
Before creating new code, check these files for existing patterns:
- `src/lib/utils.ts` - Utility functions (don't duplicate)
- `src/types/woocommerce.ts` - Type definitions (extend, don't recreate)
- `src/components/ui/` - Reusable UI components

### 3. Verify Dependencies
All necessary packages are installed. Avoid adding new dependencies unless absolutely necessary. Current stack handles:
- Forms: React Hook Form + Zod
- Animation: Framer Motion
- Styling: Tailwind CSS
- State: Zustand
- Dates: date-fns

## Code Standards

### TypeScript
```typescript
// ✅ Good: Explicit types, use existing types
import { WCProduct } from '@/types/woocommerce'
const product: WCProduct = await productsApi.get(id)

// ❌ Bad: Using 'any' or inline object types
const product: any = await fetch(...)
```

### Component Structure
```typescript
// Client component example
'use client'

import { useState } from 'react'
import { cn } from '@/lib/utils'
import { Button } from '@/components/ui/button'

interface Props {
  title: string
  className?: string
}

export function MyComponent({ title, className }: Props) {
  const [state, setState] = useState(false)

  return (
    <div className={cn('base-styles', className)}>
      <h2>{title}</h2>
      <Button onClick={() => setState(!state)}>Toggle</Button>
    </div>
  )
}
```

### Server Component with Data Fetching
```typescript
// Server component example (no 'use client')
import { productsApi } from '@/lib/woocommerce'
import { ProductGrid } from '@/components/product/product-grid'

export default async function ShopPage() {
  const products = await productsApi.list({ per_page: 12 })

  return (
    <main className="container py-8">
      <ProductGrid products={products} />
    </main>
  )
}
```

### Zustand Store Usage
```typescript
// ✅ Good: Use selector hooks to prevent re-renders
import { useCartItems, useCartTotal } from '@/stores/cart-store'

function CartSummary() {
  const items = useCartItems()
  const total = useCartTotal()
  // Component only re-renders when items or total change
}

// ❌ Bad: Selecting entire store
import { useCartStore } from '@/stores/cart-store'

function CartSummary() {
  const store = useCartStore() // Re-renders on ANY store change
}
```

### Form Handling
```typescript
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be 8+ characters'),
})

type FormData = z.infer<typeof schema>

export function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  })

  const onSubmit = async (data: FormData) => {
    // Handle submission
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  )
}
```

## API Integration Guidelines

### GraphQL Queries
```typescript
import { graphqlFetch } from '@/lib/graphql'

// Define query with variables
const query = `
  query GetProduct($slug: String!) {
    product(id: $slug, idType: SLUG) {
      id
      name
      description
    }
  }
`

// Execute with caching
const data = await graphqlFetch(query, { slug }, {
  next: { revalidate: 60 } // Cache for 60 seconds
})
```

### WooCommerce REST API
```typescript
import { productsApi, ordersApi } from '@/lib/woocommerce'

// List products with filters
const products = await productsApi.list({
  category: '5',
  per_page: 20,
  orderby: 'date',
  order: 'desc',
})

// Get single product with variations
const product = await productsApi.getBySlug('product-slug')
const variations = await productsApi.getVariations(product.id)

// Create order (server-side only)
const order = await ordersApi.create({
  payment_method: 'cod',
  billing: { /* address */ },
  line_items: [{ product_id: 123, quantity: 2 }],
})
```

### Error Handling Pattern
```typescript
import { WooCommerceError } from '@/lib/woocommerce'

try {
  const product = await productsApi.get(id)
} catch (error) {
  if (error instanceof WooCommerceError) {
    if (error.status === 404) {
      notFound() // Next.js not found page
    }
    console.error(`WC Error ${error.status}: ${error.message}`)
  }
  throw error
}
```

## Styling Guidelines

### Tailwind CSS Patterns
```typescript
// Use cn() for conditional classes
import { cn } from '@/lib/utils'


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wpacademy/nextjs-woocommerce-frontend](https://github.com/wpacademy/nextjs-woocommerce-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
