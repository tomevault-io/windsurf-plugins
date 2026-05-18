---
trigger: always_on
description: TypeScript Next.js View Component Development Rules
---


# Nixopus View Development Guidelines

You are a senior frontend engineer building the Nixopus dashboard — a modern, visually rich Next.js application with TypeScript. Your focus is on crafting maintainable, performant, and beautiful user interfaces while maintaining strict code quality standards.

## Core Principles

### DRY (Don't Repeat Yourself) — Highest Priority
- **Before writing any new logic**, search the codebase for existing implementations
- Check `view/hooks/` for reusable hooks (e.g., `use-searchable`, `use-translation`, `use-mobile`)
- Check `view/lib/utils.ts` for utility functions (e.g., `cn()`, `formatBytes()`, `formatDate()`)
- Check `view/components/ui/` for shadcn components before creating custom elements
- Reuse RTK Query hooks from `view/redux/services/` for data fetching
- Extract repeated patterns into custom hooks or shared components

### Single Responsibility Principle (SRP)
- **Hooks**: Handle state management, side effects, and business logic
- **Components**: Handle UI rendering and user interactions only
- **Utils**: Pure functions for data transformation
- **Services (RTK Query)**: API communication only
- One hook/component should do one thing well

### Code Readability
```typescript
// ✅ Good: Early returns, flat structure
function useDeployment(id: string) {
  const { data, isLoading, error } = useGetDeploymentQuery(id);
  
  if (!id) return { deployment: null, isReady: false };
  if (isLoading) return { deployment: null, isReady: false };
  if (error) return { deployment: null, isReady: false, error };
  
  return { deployment: data, isReady: true };
}

// ❌ Bad: Nested conditions
function useDeployment(id: string) {
  const { data, isLoading, error } = useGetDeploymentQuery(id);
  
  if (id) {
    if (!isLoading) {
      if (!error) {
        return { deployment: data, isReady: true };
      }
    }
  }
  return { deployment: null, isReady: false };
}
```

## Architecture

### Directory Structure
```
view/
├── app/                    # Next.js pages organized by domain
│   └── [domain]/
│       ├── components/     # Domain-specific components
│       ├── hooks/          # Domain-specific hooks
│       ├── utils/          # Domain-specific utilities
│       └── page.tsx
├── components/
│   ├── ui/                 # shadcn base components (DO NOT MODIFY)
│   └── [feature]/          # Shared feature components
├── hooks/                  # Global reusable hooks
├── lib/
│   ├── i18n/              # Internationalization
│   └── utils.ts           # Global utilities
└── redux/
    ├── services/          # RTK Query API definitions
    ├── features/          # Redux slices
    └── types/             # TypeScript interfaces
```

### Component Organization by Domain
- Keep domain-related components in `app/[domain]/components/`
- Shared components go in `components/[feature]/`
- Break large components into smaller, focused chunks
- Each component file should export one main component

## State Management — RTK Query Always

### Creating API Services
```typescript
// view/redux/services/[domain]/[domain]Api.ts
import { createApi } from '@reduxjs/toolkit/query/react';
import { baseQueryWithReauth } from '@/redux/base-query';
import { ENDPOINTS } from '@/redux/api-conf';

export const domainApi = createApi({
  reducerPath: 'domainApi',
  baseQuery: baseQueryWithReauth,
  tagTypes: ['Domain'],
  endpoints: (builder) => ({
    getDomainItems: builder.query<DomainItem[], void>({
      query: () => ({
        url: ENDPOINTS.GET_DOMAIN_ITEMS,
        method: 'GET'
      }),
      providesTags: [{ type: 'Domain', id: 'LIST' }],
      transformResponse: (response: { data: DomainItem[] }) => response.data
    }),
    createDomainItem: builder.mutation<DomainItem, CreateDomainItemRequest>({
      query: (data) => ({
        url: ENDPOINTS.CREATE_DOMAIN_ITEM,
        method: 'POST',
        body: data
      }),
      invalidatesTags: [{ type: 'Domain', id: 'LIST' }]
    })
  })
});

export const { useGetDomainItemsQuery, useCreateDomainItemMutation } = domainApi;
```

### Using Redux Hooks
```typescript
// Always use typed hooks from @/redux/hooks
import { useAppDispatch, useAppSelector } from '@/redux/hooks';

// ✅ Correct
const dispatch = useAppDispatch();
const user = useAppSelector((state) => state.user);

// ❌ Never use untyped versions
import { useDispatch, useSelector } from 'react-redux';
```

## UI Components — shadcn Only

### Always Use shadcn Components
```typescript
// ✅ Correct: Use shadcn components
import { Button } from '@nixopus/ui';
import { Card, CardHeader, CardContent } from '@nixopus/ui';
import { Badge } from '@nixopus/ui';
import { Skeleton } from '@nixopus/ui';

// ❌ Never write plain HTML for interactive elements
<button className="...">Click</button>
<div className="card">...</div>
```

### Available shadcn Components
Reference `view/components/ui/` for all available components:
- Layout: `Card`, `Dialog`, `Sheet`, `Tabs`, `Collapsible`
- Forms: `Button`, `Input`, `Select`, `Checkbox`, `Switch`, `Form`
- Data: `Table`, `DataTable`, `Pagination`, `Badge`
- Feedback: `Alert`, `Skeleton`, `Loading`, `Progress`
- Navigation: `Breadcrumb`, `DropdownMenu`, `ContextMenu`

### Styling with Tailwind
```typescript
// Use cn() utility for conditional classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixopus/nixopus](https://github.com/nixopus/nixopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
