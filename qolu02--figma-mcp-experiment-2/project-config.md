---
trigger: always_on
description: This project demonstrates Claude Code's ability to transpose Figma designs into production-quality React TypeScript components. Each Figma mockup is implemented as a separate route, showcasing component composition, design system adherence, and responsive layouts.
---

# Figma to React Showcase

## Overview

This project demonstrates Claude Code's ability to transpose Figma designs into production-quality React TypeScript components. Each Figma mockup is implemented as a separate route, showcasing component composition, design system adherence, and responsive layouts.

**Tech Stack:**
- Vite
- React 18+
- TypeScript
- Tailwind CSS
- Shadcn UI
- React Router
- Recharts

**Objective:** Demonstrate agentic coding workflows: observe Figma designs via MCP, decompose into semantic component hierarchies, and implement with best practices.

---

## TypeScript Standards

### Critical Rules
- **NEVER use `any` type** - use `unknown` if type is genuinely uncertain
- Enable `strict: true` in tsconfig.json
- Always define explicit prop interfaces
- Use discriminated unions for variants: `type Status = 'idle' | 'loading' | 'error'`
- Prefer `interface` for object shapes, `type` for unions/intersections

### Component Types
```typescript
// ✅ Correct
import { useState } from 'react'; // Only import what you use (hooks, types)

interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({ label, variant = 'primary', onClick }) => {
  // Implementation
};

// ❌ Never do this
export const Button = (props: any) => { /* ... */ };

// Note: React 18+ JSX transform - no need for `import React from 'react'`
```

### Type Organization
- **Component props** → Define inline in component file
- **Domain/data models** → Extract to route-specific `types/` folder (Product, Order, etc.)

```typescript
// routes/shop/types/product.ts - Route-specific domain model
export interface Product {
  id: number;
  title: string;
  price: number;
  image: string;
}

// routes/shop/components/organisms/ProductGrid.tsx - Props inline, import domain model
import type { Product } from '../../types/product';

interface ProductGridProps {  // Inline - component-specific
  products: Product[];
  onProductClick?: (id: number) => void;
}

export const ProductGrid: React.FC<ProductGridProps> = ({ products }) => { };
```

---

## Project Structure

```
src/
├── routes/                 # ALL routes (complete isolation)
│   ├── home/               # Navigation hub route
│   │   └── HomePage.tsx    # Simple page - no components/ needed
│   ├── shop/
│   │   ├── components/
│   │   │   ├── molecules/ # ProductCard, SearchBar, HeaderAuth
│   │   │   └── organisms/ # Header, FilterMenu, ProductGrid, Footer
│   │   ├── types/
│   │   │   └── product.ts # Route-specific types
│   │   └── ShopPage.tsx   # Page component
│   └── dashboard/
│       ├── components/
│       │   ├── molecules/ # StatCard, DonutChart, BarChart
│       │   └── organisms/ # Sidebar, TopNav, RecentOrders
│       ├── types/
│       │   └── order.ts   # Route-specific types
│       └── DashboardPage.tsx
├── components/
│   └── ui/                 # Shadcn ONLY (auto-generated, required)
├── lib/                    # Utilities (utils.ts, cn function)
├── App.tsx                 # Router configuration
└── main.tsx               # Entry point
```

### File Organization Rules
- **Pure route isolation**: Each route is completely self-contained with components, types, and page in `routes/[route-name]/`
- **No shared components**: Import Shadcn components directly from `@/components/ui/` (no wrappers needed)
- **Only create what you need**:
  - Simple pages (like home) → Just the page file, no components/
  - Complex pages → Create components/ and types/ as needed
  - Custom hooks → Create in route-specific hooks/ directory (not top-level)
  - Atoms → Only if route needs custom wrappers (otherwise import Shadcn directly)
- **Route imports**: Use relative imports within route (`./components/molecules/ProductCard`)
- **Shadcn imports**: Use @ alias to import directly (`@/components/ui/button`)
- **Single component export** → Flat file
- **Multiple related components** → Folder with index.ts

```typescript
// Route-specific component: routes/shop/components/molecules/ProductCard.tsx
import type { Product } from '../../types/product'; // Relative import within route
import { Button } from '@/components/ui/button';     // Direct Shadcn import

export const ProductCard: React.FC<{ product: Product }> = ({ product }) => {
  // Implementation
};

// If route needs custom hook, create route-specific hooks/:
// routes/shop/hooks/useProductFilter.ts
export const useProductFilter = () => {
  // Route-specific hook logic
};

// If route needs custom wrapper, create route-specific atoms/:
// routes/shop/components/atoms/ShopButton.tsx
import { Button as ShadcnButton } from '@/components/ui/button';

export const ShopButton: React.FC = (props) => (
  <ShadcnButton className="bg-shop-primary" {...props} />
);
```

### Path Aliases (Required)
Configure in `vite.config.ts` and `tsconfig.json`:
```typescript
// vite.config.ts
import path from 'path';
export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});

// tsconfig.json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qolu02/figma_mcp_experiment_2](https://github.com/qolu02/figma_mcp_experiment_2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
