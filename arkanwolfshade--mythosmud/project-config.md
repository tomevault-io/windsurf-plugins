---
trigger: always_on
description: This guide defines definitive best practices for Next.js applications, focusing on the `app` directory, Server Components, performance, and maintainability. Follow these rules to build robust, scalable, and performant Next.js projects.
---

# next-js Best Practices

This document outlines the definitive best practices for developing Next.js applications. Adhering to these guidelines ensures consistent, performant, and maintainable code, leveraging Next.js's strengths for modern web development.

## 1. Code Organization and Structure

Always use the `app/` directory for new projects. Organize code by feature, not by type, to improve discoverability and cohesion.

### ✅ GOOD: Feature-Driven `app/` Directory

Group all related files for a feature (components, pages, layouts, hooks, types) within a single directory.

```tsx
// app/dashboard/page.tsx
export default function DashboardPage() { /* ... */ }

// app/dashboard/layout.tsx
export default function DashboardLayout({ children }) { /* ... */ }

// app/dashboard/components/DashboardOverview.tsx
export function DashboardOverview() { /* ... */ }

// app/dashboard/hooks/useDashboardData.ts
export function useDashboardData() { /* ... */ }
```

### ❌ BAD: Type-Driven `app/` Directory

Avoid scattering files of the same feature across different top-level type directories.

```tsx
// app/dashboard/page.tsx
// components/dashboard/DashboardOverview.tsx // Separated
// hooks/useDashboardData.ts // Separated
```

### Core Directories

*   **`app/`**: All route-related files (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `route.ts`).
*   **`components/`**: Reusable UI components that are *not* directly tied to a specific route.
*   **`lib/`**: Backend-agnostic utility functions, data access layers, and third-party integrations.
*   **`hooks/`**: Custom React hooks for reusable logic.
*   **`types/`**: Global TypeScript type definitions and interfaces.
*   **`public/`**: Static assets (images, fonts) that are served directly.

## 2. Component Architecture: Server Components First

Prioritize Server Components for all UI rendering. Use Client Components only when interactivity (state, effects, event handlers) is strictly required.

### ✅ GOOD: Server Component by Default

Server Components reduce client-side JavaScript bundles, improve initial page load, and enhance security by keeping sensitive logic on the server.

```tsx
// app/products/[id]/page.tsx (Server Component by default)
import { getProductDetails } from '@/lib/api';

export default async function ProductPage({ params }) {
  const product = await getProductDetails(params.id);
  return (
    <div>
      <h1>{product.name}</h1>
      {/* ... more server-rendered UI */}
      <AddToCartButton productId={product.id} /> {/* Client Component */}
    </div>
  );
}
```

### ✅ GOOD: "use client" Boundary as Low as Possible

Place the `"use client"` directive at the lowest possible point in your component tree. This ensures that only the interactive parts are client-rendered, keeping parent components as Server Components.

```tsx
// components/AddToCartButton.tsx
'use client'; // Only this component and its children are client-side

import { useState } from 'react';

export function AddToCartButton({ productId }) {
  const [quantity, setQuantity] = useState(1);
  // ... interactive logic
  return <button onClick={() => alert(`Added ${quantity} of ${productId}`)}>Add to Cart</button>;
}
```

### ❌ BAD: Overuse of "use client"

Don't mark entire feature folders or layouts as client components if only a small part needs interactivity. This unnecessarily increases client bundle size.

```tsx
// app/products/[id]/page.tsx (BAD: entire page marked client)
'use client'; // This makes the whole page a client component
import { useState, useEffect } from 'react'; // Even if only a small part needs it

export default function ProductPage({ params }) {
  // ...
  return <button>Add to Cart</button>;
}
```

## 3. Data Fetching

Fetch data directly in Server Components using `fetch` or a dedicated data access layer. Use Route Handlers for client-side mutations or when exposing a specific API endpoint.

### ✅ GOOD: Server Component Data Fetching

Directly fetch data in Server Components. `fetch` requests are automatically memoized and cached by Next.js.

```tsx
// app/dashboard/page.tsx
import { getUserProfile, getRecentOrders } from '@/lib/api';

export default async function DashboardPage() {
  // Data fetches in parallel
  const [user, orders] = await Promise.all([
    getUserProfile(),
    getRecentOrders(),
  ]);

  return (
    <div>
      <h2>Welcome, {user.name}</h2>
      <OrderList orders={orders} />
    </div>
  );
}
```

### ✅ GOOD: Route Handlers for Client-Side Mutations

Use `route.ts` for API endpoints that handle client-side data mutations (e.g., form submissions, API calls from client components).

```tsx
// app/api/cart/route.ts
import { NextResponse } from 'next/server';
import { addToCart } from '@/lib/cart';

export async function POST(request: Request) {
  const { productId, quantity } = await request.json();
  await addToCart(productId, quantity);
  return NextResponse.json({ success: true });
}
```

### ✅ GOOD: Streaming with `loading.tsx` and `Suspense`

Improve perceived performance by showing instant loading states for slow data fetches.

```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arkanwolfshade) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
