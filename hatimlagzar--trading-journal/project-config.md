---
trigger: always_on
description: A Next.js 16 trading journal application using Supabase for authentication and database. Built with TypeScript, React 19, and Tailwind CSS v4.
---

# AGENTS.md - Trade In Systems Development Guide

## Project Overview

A Next.js 16 trading journal application using Supabase for authentication and database. Built with TypeScript, React 19, and Tailwind CSS v4.

## Build, Lint, and Test Commands

```bash
# Development
npm run dev              # Start Next.js development server (localhost:3000)

# Production
npm run build            # Build for production
npm run start            # Start production server
npm run lint             # Run ESLint on the codebase

# No test framework is currently configured
```

## Code Style Guidelines

### TypeScript
- Strict mode is enabled in `tsconfig.json`
- Always use explicit types for function parameters and return types
- Use `import type` for type-only imports to improve performance

```typescript
// Good
import { supabase } from './supabase/client'
import type { TradeInsert, TradeUpdate } from './types'

// Bad
import { supabase, type TradeInsert } from './supabase/client'
```

### React/Next.js Conventions
- Use `'use client'` directive for client-side components
- Use functional components with TypeScript interfaces for props
- Name components using PascalCase
- Use file-based routing (App Router)
- Prefer server page wrappers for authenticated dashboards when initial data can be fetched on the server; pass initial props into a client component for interactive state

```typescript
interface TradeFormProps {
  trade?: Trade | null
  onClose: () => void
  onSuccess: () => void
  userId: string
}

export default function TradeForm({ trade, onClose, onSuccess, userId }: TradeFormProps) {
  // ...
}
```

### Naming Conventions
- Components: PascalCase (e.g., `TradeForm`, `TradeModal`)
- Functions/variables: camelCase (e.g., `getTrades`, `formData`)
- Types: PascalCase (e.g., `Trade`, `TradeInsert`, `System`, `SubSystem`)
- File names: kebab-case for non-components (e.g., `trade-form.tsx`, `trades.ts`)

### Imports Order
1. React/Next imports
2. External libraries (Supabase, etc.)
3. Internal imports (lib/, app/)
4. Type imports

```typescript
'use client'

import { useState, useEffect, useRef } from 'react'
import { createTrade, updateTrade } from '@/services/trade'
import { uploadScreenshot } from '@/services/upload'
import type { Trade, TradeInsert } from '@/services/trade'
```

### Path Aliases
Use `@/*` for absolute imports:
```typescript
import { getTrades } from '@/services/trade'
import { uploadScreenshot } from '@/services/upload'
import type { Trade } from '@/services/trade'
```

### Styling
- Use Tailwind CSS v4 utility classes
- Prefer semantic HTML elements
- Use `className` for Tailwind classes
- Ensure clickable buttons show pointer cursor (`cursor-pointer`) unless disabled

### Error Handling
- Throw Supabase errors directly after operations
- Catch errors in async handlers and set error state
- Display user-friendly error messages in UI

```typescript
// In data access functions
if (error) throw error

// In event handlers
try {
  await createTrade(formData)
  onSuccess()
} catch (err) {
  setError(err instanceof Error ? err.message : 'Failed to save trade')
}
```

### State Management
- Use `useState` for local component state
- Use Context API for global state (see `lib/AuthContext.tsx`)
- Prefer functional updates: `setFormData(prev => ({ ...prev, field: value }))`

### Database/Supabase
- Define types in respective service's `types.ts` (e.g., `services/trade/types.ts`)
- Use `TradeInsert` and `TradeUpdate` types for create/update operations
- `trades` supports both `system_id` and `sub_system_id` assignments
- Always filter queries by `user_id` for multi-user support

### ESLint Configuration
- Uses `eslint-config-next` with TypeScript support
- Run `npm run lint` before committing

### Formatting
- Use single quotes for strings
- Use 2-space indentation
- Add trailing commas in objects and arrays
- Use semicolons at the end of statements

### General Patterns
- Place related functions in dedicated module files (e.g., `services/trade/trades.ts`)
- Use section comments for grouping code (optional)
- Initialize optional fields with `null` rather than `undefined`
- Use `URL.revokeObjectURL()` to clean up object URLs in useEffect cleanup

## Project Structure

```
/app              # Next.js App Router pages
  /backtesting    # Backtesting sessions + theoretical trades page
    ImportBacktestingTradesForm.tsx # Backtesting sheet import modal with mapping preview
  /trades         # Trades page and components
    CloseTradeForm.tsx # Quick close-trade modal form (win/loss P&L input)
    ImportTradesForm.tsx # Sheet/CSV import modal with mapping preview
    /[tradeId]/page.tsx # Per-trade focus workspace (thinking discussion + chart snippet)
  /systems        # Systems + sub-systems management page
  /settings       # Account settings page (password update)
  /login          # Login page
  /signup         # Signup page
  layout.tsx      # Root layout
  page.tsx        # Home page
/lib              # Shared client-side code
  /supabase       # Supabase client setup
  AuthContext.tsx # Authentication context
/services         # Backend logic (Supabase operations)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HatimLagzar/trading-journal](https://github.com/HatimLagzar/trading-journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
