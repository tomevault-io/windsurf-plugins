---
trigger: always_on
description: Follow these rules when working on the frontend.
---

# Frontend Architecture Guidelines

## Technology Stack

Our frontend leverages a modern stack designed for performance, developer experience, and maintainability:

- **Framework**: Next.js 15+ with App Router
- **Styling**: Tailwind CSS for utility-first styling
- **UI Components**: Shadcn UI as a component foundation
- **Animations**: Framer Motion for fluid animations
- **Icons**: Lucide React for consistent iconography

## Core Principles

1. **Type Safety**: Leverage TypeScript for robust, maintainable code
2. **Component-Driven Development**: Build reusable, composable components 
3. **Performance-First**: Optimize for core web vitals and user experience
4. **Accessibility**: Ensure WCAG compliance in all UIs
5. **Progressive Enhancement**: Build resilient interfaces that work across devices

## Component Architecture

### Component Types

Our application uses both **Server Components** and **Client Components** following Next.js paradigms:

| Component Type | Use Case | Key Considerations |
|---------------|----------|-------------------|
| **Server Components** | Data fetching, SEO-critical content | No `useState`, no event handlers |
| **Client Components** | Interactive UIs, forms, animations | Add `"use client"` directive at top of file |

### Component Organization

```
app/
├── [route]/              # Route directories
│   ├── _components/      # Route-specific components
│   ├── page.tsx          # Route page component
│   └── layout.tsx        # Route layout component
components/
├── ui/                   # Shared UI components
└── [feature]/            # Feature-specific shared components
```

#### Naming Conventions

- Use kebab-case for all component files: `data-table.tsx` 
- Use PascalCase for component names: `DataTable`
- Group related components in feature-specific directories
- Suffix context providers with `Provider`: `AuthProvider`

### Component Guidelines

#### Server vs Client Components

- Always add the appropriate directive at the top of your file:
  - `"use server"` for server components
  - `"use client"` for client components

- Server components should:
  - Perform data fetching
  - Pass data to client components via props
  - Minimize client-side JavaScript
  
- Client components should:
  - Handle user interactions
  - Manage local state
  - Implement animations and effects

#### Code Structure

- Import ordering:
  1. React and Next.js imports
  2. Third-party libraries
  3. Internal components and utilities
  4. Types and styles

- Export components as named exports for components meant to be used within a directory, and as default exports for page components or major feature components

## Server Component Patterns

### Data Fetching

- Fetch data directly in server components using appropriate patterns:

```tsx
// Good: Fetch in server components
async function ProductList() {
  const products = await getProductsAction()

  return <ProductGrid products={products.data} />
}
```

- Use Suspense boundaries for asynchronous content:

```tsx
// page.tsx
export default function Page() {
  return (
    <div>
      <Header />
      <Suspense fallback={<ProductsSkeleton />}>
        <ProductsContent />
      </Suspense>
    </div>
  )
}

// Async component in same file or imported
async function ProductsContent() {
  const products = await getProducts()

  return <ProductList products={products.data} />
}
```

### Detailed Examples

#### Server Layout Example

```tsx
// app/dashboard/layout.tsx
import { SidebarNav } from "./_components/sidebar-nav"
import { DashboardHeader } from "./_components/dashboard-header"
import { getProfile } from "@/actions/profile"
import { redirect } from "next/navigation"

export default async function DashboardLayout({
  children
}: {
  children: React.ReactNode
}) {
  const profile = await getProfile()
  
  if (!profile.isSuccess) {
    return redirect("/login")
  }
  
  return (
    <div className="flex min-h-screen flex-col">
      <DashboardHeader user={profile.data} />
      
      <div className="flex flex-1">
        <SidebarNav />
        <main className="flex-1 p-6">
          {children}
        </main>
      </div>
    </div>
  )
}
```

#### Server Page with Suspense

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react"
import { DashboardSkeleton } from "./_components/dashboard-skeleton"
import { DashboardMetrics } from "./_components/dashboard-metrics"
import { DashboardCharts } from "./_components/dashboard-charts"

export default function DashboardPage() {
  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Dashboard</h1>
      
      <Suspense fallback={<DashboardSkeleton type="metrics" />}>
        <DashboardMetricsContent />
      </Suspense>
      
      <Suspense fallback={<DashboardSkeleton type="charts" />}>
        <DashboardChartsContent />
      </Suspense>
    </div>
  )
}

async function DashboardMetricsContent() {
  const metrics = await getMetrics()

  return <DashboardMetrics data={metrics.data} />
}

async function DashboardChartsContent() {
  const chartData = await getChartData()

  return <DashboardCharts data={chartData.data} />
}
```

## Client Component Patterns

### State Management

- Use React's built-in state management for component-level state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
