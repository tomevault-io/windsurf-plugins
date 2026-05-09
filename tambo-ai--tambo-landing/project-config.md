---
trigger: always_on
description: description: Architecture patterns, state management, routing, and best practices
---

---
description: Architecture patterns, state management, routing, and best practices
globs: *.tsx, *.jsx, *.css, *.js, *.ts
---

# Architecture Guidelines

## Type Safety

### TypeScript Configuration
- Use TypeScript for all new code
- Maintain strict type checking
- Avoid `any` types unless absolutely necessary
- Use proper type imports (`import type` when importing only types)

```tsx
import type { ComponentProps } from 'react'

interface ButtonProps extends ComponentProps<'button'> {
  variant?: 'primary' | 'secondary'
}
```

## State Management

### React Built-in State
Prefer React's built-in state for component state. Keep state as close to where it's used as possible.

```tsx
function Component() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### Zustand for Global State
Use Zustand for global state when needed. Define stores in `~/libs/store.ts` or dedicated store files.

```tsx
import { create } from 'zustand'

interface CartStore {
  items: CartItem[]
  addItem: (item: CartItem) => void
  removeItem: (id: string) => void
}

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({ 
    items: state.items.filter(item => item.id !== id) 
  })),
}))
```

### State Management Best Practices
- Keep state minimal and derived values computed
- Use context for shared UI state (theme, modals)
- Use Zustand for complex global state (cart, user)
- Avoid prop drilling with composition patterns

## Routing & Navigation

### Next.js App Router
Use Next.js App Router conventions. Follow the file-based routing structure.

```
app/
  (pages)/
    home/
      page.tsx
    about/
      page.tsx
```

### Navigation
Use the custom Link component for internal navigation. It automatically handles external links.

```tsx
import Link from '~/components/link'

function Navigation() {
  return (
    <>
      {/* Internal link - uses next/link */}
      <Link href="/about">About</Link>
      
      {/* External link - uses <a> */}
      <Link href="https://example.com">External</Link>
    </>
  )
}
```

### Metadata & SEO
Use `~/libs/metadata` for SEO optimization. Generate metadata for all pages.

```tsx
import { generateMetadata as generateBaseMetadata } from '~/libs/metadata'

export async function generateMetadata({ params }) {
  const page = await fetchPage(params.slug)
  
  return generateBaseMetadata({
    title: page.title,
    description: page.description,
    image: page.image,
    url: `/pages/${params.slug}`,
  })
}
```

### Loading and Error States
Implement proper loading and error states for all routes.

```tsx
// loading.tsx
export default function Loading() {
  return <div>Loading...</div>
}

// error.tsx
'use client'

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

## Performance

### Server Components
Use React Server Components by default. Only add 'use client' when needed.

```tsx
// Server Component (default)
async function ServerComponent() {
  const data = await fetchData()
  return <div>{data.title}</div>
}

// Client Component (when needed)
'use client'

function ClientComponent() {
  const [state, setState] = useState(0)
  return <button onClick={() => setState(state + 1)}>{state}</button>
}
```

### Code Splitting
Use `next/dynamic` for heavy components. Implement proper loading states.

```tsx
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div>Loading...</div>,
  ssr: false // if needed
})
```

### Caching Strategies
Follow Next.js 16 recommended caching strategies. Use appropriate revalidation times.

```tsx
// Static generation with revalidation
export const revalidate = 3600 // 1 hour

// Dynamic with specific cache tags
export async function fetchData() {
  const res = await fetch('https://api.example.com/data', {
    next: { 
      revalidate: 3600,
      tags: ['data']
    }
  })
  return res.json()
}

// User-specific data - NEVER cache
export async function fetchUserCart(userId: string) {
  const res = await fetch(`https://api.example.com/cart/${userId}`, {
    cache: 'no-store' // Required for user-specific data
  })
  return res.json()
}
```

### Cache Components (Next.js 16)

Cache Components are enabled globally (`cacheComponents: true`). Key considerations:

**Suspense Boundaries:**
```tsx
import { Suspense } from 'react'

export default async function Page() {
  return (
    <Suspense fallback={<Loading />}>
      <DataComponent />
    </Suspense>
  )
}
```

**Cache Invalidation:**
```tsx
import { revalidateTag, revalidatePath } from 'next/cache'

// In webhook handlers
export async function POST(request: Request) {
  revalidateTag('products')
  // or
  revalidatePath('/products/[slug]', 'page')
  return Response.json({ revalidated: true })
}
```

**⚠️ Critical Rules:**
- User-specific data: Always use `cache: 'no-store'`
- Real-time data: Always use `cache: 'no-store'`
- Test with hard refresh AND navigation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tambo-ai/tambo-landing](https://github.com/tambo-ai/tambo-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
