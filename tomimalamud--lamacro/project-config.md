---
trigger: always_on
description: **Lamacro** is a Next.js 15 financial data platform focused on Argentine financial markets. This system integrates with the BCRA (Central Bank of Argentina) API and provides tools for analyzing government bonds, inflation data, stock performance, debt information, and financial calculations.
---

# Lamacro - Next.js 15 Financial Platform Rules

## Project Overview

**Lamacro** is a Next.js 15 financial data platform focused on Argentine financial markets. This system integrates with the BCRA (Central Bank of Argentina) API and provides tools for analyzing government bonds, inflation data, stock performance, debt information, and financial calculations.

**Core Architecture**: Server-first Next.js 15 App Router with TypeScript, Redis caching, and external API integration.

## Next.js 15 App Router Patterns

### Server-First Architecture
- **Default to Server Components**: All components should be Server Components unless client-side interactivity is required
- **Data fetching**: Always perform data fetching in Server Components via [lib/actions.ts](mdc:lib/actions.ts) and [lib/tamar-actions.ts](mdc:lib/tamar-actions.ts)
- **Minimize 'use client'**: Only use `'use client'` directive for:
  - Event handlers (onClick, onChange, onSubmit)
  - Browser APIs (localStorage, document, window)
  - React hooks (useState, useEffect, useContext)
  - Third-party libraries requiring client-side execution

### File Structure Conventions
```
app/
├── [feature]/              # Route groups for features
│   ├── page.tsx           # Server Component (data fetching)
│   ├── loading.tsx        # Streaming UI component
│   └── error.tsx          # Error boundary
components/
├── [feature]/             # Domain-specific components
│   ├── [feature]-client.tsx  # Client components (use client)
│   └── [feature]-table.tsx   # Server components
lib/
├── [feature].ts          # Business logic modules
└── actions.ts            # Server actions
```

### Component Patterns
- **Server Components**: Use for data fetching, layout, and static content
- **Client Components**: Keep as leaf nodes in component tree when possible
- **Client Component Naming**: Suffix client components with `-client.tsx` (see [components/carry-trade/carry-trade-client.tsx](mdc:components/carry-trade/carry-trade-client.tsx))
- **Composition over 'use client'**: Use component composition to avoid marking parent components as Client Components

## Business Logic Architecture

### Domain-Driven Organization
Core business modules are organized by financial domain in [lib/](mdc:lib/):

- **[acciones.ts](mdc:lib/acciones.ts)** - Stock market analysis with inflation-adjusted returns
- **[carry-trade.ts](mdc:lib/carry-trade.ts)** - Government bond arbitrage analysis with MEP calculations  
- **[duales.ts](mdc:lib/duales.ts)** - Dual currency bond analysis with scenario modeling
- **[fija.ts](mdc:lib/fija.ts)** - Fixed income securities with yield calculations (TNA, TEM, TEA)
- **[debts.ts](mdc:lib/debts.ts)** - Central Bank debt registry integration

### API Integration Pattern
```typescript
External APIs → bcra-api-helper → bcra-fetch → Domain Logic → UI Components
                      ↓
                 redis-cache (fallback)
```

Key files:
- **[bcra-api-helper.ts](mdc:lib/bcra-api-helper.ts)** - Circuit breaker pattern, rate limiting, error handling
- **[bcra-fetch.ts](mdc:lib/bcra-fetch.ts)** - BCRA API integration with fallback caching
- **[redis-cache.ts](mdc:lib/redis-cache.ts)** - 7-day TTL fallback for BCRA data

### Server Actions Pattern
- **[actions.ts](mdc:lib/actions.ts)** - Primary server actions for UI-server bridge
- **[tamar-actions.ts](mdc:lib/tamar-actions.ts)** - Specialized server actions for external integrations
- Always use server actions for data mutations and form handling
- Never perform data fetching in Client Components

## TypeScript Conventions

### Type Organization
- **Domain types**: [types/](mdc:types/) directory for complex business types
- **Interface over type**: Prefer interfaces for object definitions
- **No TypeScript enums**: Use constant objects/maps instead
- **No 'any' types**: Always provide proper type definitions

### Utility Functions
Common utilities in [lib/utils.ts](mdc:lib/utils.ts):
- `cn()` - Tailwind class merging with clsx
- `formatNumber()` - Argentine locale number formatting  
- `formatDateAR()` - Argentine date formatting
- `getNextBusinessDay()` - Financial business day calculations

## UI and Styling Patterns

### Component Library Strategy
- **shadcn/ui**: Primary UI component library (see [components/ui/](mdc:components/ui/))
- **Lucide React**: Icon library over custom SVGs
- **Never recreate shadcn components**: Always use existing or add new ones via CLI

### Styling Conventions
- **Tailwind CSS**: Primary styling approach
- **Design tokens**: Consistent spacing, colors, typography
- **Dark/light mode**: Support via [next-themes](mdc:app/layout.tsx)
- **Responsive design**: Mobile-first approach

### Layout Pattern
Root layout in [app/layout.tsx](mdc:app/layout.tsx):
- Theme provider with system detection
- Navigation and footer components
- PostHog analytics integration
- Toast notifications via Sonner

## Performance and Caching

### Caching Strategy
- **In-memory cache**: Short-term API response caching
- **Redis fallback**: 7-day TTL for BCRA API failures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TomiMalamud/lamacro](https://github.com/TomiMalamud/lamacro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
