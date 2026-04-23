---
trigger: always_on
description: This file provides comprehensive guidance for AI assistants and developers working on LibreDB Studio. Follow these rules to maintain code quality, consistency, and architectural integrity.
---

# LibreDB Studio - Cursor Rules

This file provides comprehensive guidance for AI assistants and developers working on LibreDB Studio. Follow these rules to maintain code quality, consistency, and architectural integrity.

## Project Overview

LibreDB Studio is a modern, AI-powered, open-source SQL IDE for cloud-native teams. It provides a web-based interface for managing PostgreSQL, MySQL, SQLite, and MongoDB databases with features like AI-assisted query generation, visual schema exploration, and real-time monitoring.

**Key Characteristics:**
- Mobile-first, professional-always design philosophy
- Zero-install browser-based solution
- Multi-database support via Strategy Pattern
- AI-native with multi-model LLM support (Gemini, OpenAI, Ollama, Custom)
- Enterprise-grade with RBAC, query auditing, and health monitoring

## Tech Stack

### Core Framework
- **Next.js 15** (App Router) with React 19
- **TypeScript** (strict mode enabled)
- **Bun** (preferred runtime) or Node.js 20+
- **Tailwind CSS 4** with `@theme inline` theming
- **shadcn/ui** components (Radix UI primitives)

### Key Libraries
- **Monaco Editor** - SQL editor (VS Code engine)
- **TanStack Table & Virtual** - Virtualized data grids
- **Framer Motion** - Animations
- **Jose** - JWT authentication
- **Zod** - Runtime type validation
- **React Hook Form** - Form management

### Database Drivers
- PostgreSQL: `pg`
- MySQL: `mysql2`
- SQLite: `better-sqlite3` (via dynamic import)
- MongoDB: `mongodb`

## Code Style & Conventions

### TypeScript

- **Always use TypeScript** - No JavaScript files unless absolutely necessary
- **Strict mode** - TypeScript strict mode is enabled, respect it
- **Avoid `any`** - Use `unknown` or proper types instead. If `any` is unavoidable, add a comment explaining why
- **Explicit types** - Prefer explicit return types for functions, especially public APIs
- **Type imports** - Use `import type` for type-only imports
- **Interfaces over types** - Prefer `interface` for object shapes, use `type` for unions, intersections, and aliases

```typescript
// ✅ Good
interface DatabaseConnection {
  id: string;
  name: string;
  type: DatabaseType;
}

type DatabaseType = 'postgres' | 'mysql' | 'sqlite' | 'mongodb' | 'redis' | 'oracle' | 'mssql';

// ❌ Bad
type DatabaseConnection = {
  id: string;
  name: string;
}
```

### Naming Conventions

- **Files**: kebab-case for components (`query-editor.tsx`), PascalCase for components (`QueryEditor.tsx`)
- **Components**: PascalCase (`QueryEditor`, `Dashboard`)
- **Functions/Variables**: camelCase (`fetchUser`, `activeConnection`)
- **Constants**: UPPER_SNAKE_CASE (`DEFAULT_TIMEOUT`, `MAX_POOL_SIZE`)
- **Types/Interfaces**: PascalCase (`DatabaseConnection`, `QueryResult`)
- **Hooks**: camelCase starting with `use` (`useToast`, `useMonitoringData`)
- **API Routes**: kebab-case (`/api/db/query`, `/api/auth/login`)

### React Patterns

- **Functional components only** - No class components
- **Hooks over HOCs** - Prefer custom hooks for shared logic
- **Server Components by default** - Use `"use client"` only when necessary (interactivity, hooks, browser APIs)
- **Component organization**:
  ```typescript
  // 1. Imports (external, then internal)
  // 2. Types/Interfaces
  // 3. Component
  // 4. Exports
  ```
- **Props destructuring** - Destructure props in function signature
- **Memoization** - Use `useMemo`/`useCallback` sparingly, only for expensive computations or stable references

```typescript
// ✅ Good
export default function QueryEditor({ query, onChange }: QueryEditorProps) {
  const handleChange = useCallback((value: string) => {
    onChange(value);
  }, [onChange]);
  
  return <MonacoEditor value={query} onChange={handleChange} />;
}

// ❌ Bad
export default function QueryEditor(props: QueryEditorProps) {
  return <MonacoEditor value={props.query} onChange={props.onChange} />;
}
```

### File Organization

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes (REST endpoints)
│   ├── (pages)/           # Page components
│   └── layout.tsx         # Root layout
├── components/            # React components
│   ├── ui/               # shadcn/ui primitives
│   └── [Feature].tsx     # Feature components
├── hooks/                 # Custom React hooks
├── lib/                   # Utilities and business logic
│   ├── db/               # Database providers (Strategy Pattern)
│   ├── llm/              # LLM providers (Strategy Pattern)
│   └── utils.ts          # General utilities
└── types.ts              # Shared TypeScript types
```

### Import Order

1. External dependencies (React, Next.js, third-party)
2. Internal absolute imports (`@/components`, `@/lib`)
3. Relative imports (`./types`, `../utils`)
4. Type-only imports last (if separate)

```typescript
// ✅ Good
import { useState } from 'react';
import { NextRequest, NextResponse } from 'next/server';
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';
import type { DatabaseConnection } from '@/lib/types';
```

## Architecture Patterns

### Strategy Pattern for Providers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libredb/libredb-studio](https://github.com/libredb/libredb-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
