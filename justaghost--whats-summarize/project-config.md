---
trigger: always_on
description: A comprehensive platform for analyzing and summarizing conversations across multiple messaging platforms, starting with WhatsApp. The application provides AI-powered summaries of chat histories with a modern, secure, and scalable architecture.
---

# WhatsApp Conversation Summarizer - AI Assistant Instructions

## Project Overview
A comprehensive platform for analyzing and summarizing conversations across multiple messaging platforms, starting with WhatsApp. The application provides AI-powered summaries of chat histories with a modern, secure, and scalable architecture.

## Architecture

This is a **monorepo** managed with `pnpm` and `Turbo`, following a modular architecture with clear separation of concerns.

### Key Principles
- **Server-Side Rendering**: Leverage Next.js App Router for optimal performance and security
- **Authentication Middleware**: Server-side authentication protection for all user routes
- **Component-Based UI**: Reusable, accessible components following shadcn/ui methodology
- **Type Safety**: Strict TypeScript across the entire codebase
- **Testing**: Comprehensive test coverage with Jest, React Testing Library, and Playwright

## Technology Stack

### Frontend
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript (strict mode)
- **UI Components**: Custom component library using Tailwind CSS and Radix UI primitives (shadcn/ui style)
- **Styling**: Tailwind CSS for utility-first styling
- **Icons**: Lucide React
- **State Management**: React Context and Server Components

### Backend
- **Platform**: Supabase
- **Database**: PostgreSQL with Row Level Security (RLS)
- **Authentication**: Supabase Auth with SSR
- **Storage**: Supabase Storage for file uploads
- **API**: Next.js API Routes and Server Actions

### Testing
- **Unit/Integration**: Jest with React Testing Library
- **E2E**: Playwright
- **Coverage**: SonarCloud integration

### DevOps
- **Package Manager**: pnpm 8.x
- **Build Tool**: Turbo (monorepo orchestration)
- **CI/CD**: GitHub Actions
- **Code Quality**: ESLint, Prettier, SonarCloud
- **Deployment**: Azure Container Apps

## Project Structure

```
.
├── apps/
│   ├── web/           # Next.js frontend application (primary app)
│   ├── api/           # Backend API service
│   ├── backend/       # Node.js backend service
│   └── frontend/      # Additional frontend service
├── packages/          # Shared packages and utilities
├── tests/             # E2E test suites (Playwright)
├── tools/             # Build and development tools
└── docs/              # Documentation
```

## Code Generation Rules

### When Creating Next.js Pages (App Router)
```typescript
// app/[route]/page.tsx
// Always use Server Components by default
export default async function PageName() {
  // Fetch data directly in the component
  const data = await fetchData();
  
  return (
    <div>
      {/* UI components */}
    </div>
  );
}

// For client components, use 'use client' directive
'use client';

export default function ClientComponent() {
  // Client-side logic
}
```

### When Creating API Routes
```typescript
// app/api/[route]/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  // Always validate authentication first
  // Handle the request
  // Return proper NextResponse
}

export async function POST(request: NextRequest) {
  // Same pattern
}
```

### When Creating UI Components
```typescript
// Always follow shadcn/ui patterns
import { cn } from '@/lib/utils';

interface ComponentProps {
  // Always define props interface
  className?: string;
  // Other props
}

export function Component({ className, ...props }: ComponentProps) {
  return (
    <div className={cn("base-classes", className)} {...props}>
      {/* Component content */}
    </div>
  );
}
```

### When Working with Supabase Auth
```typescript
// Always use SSR-compatible clients
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export async function createClient() {
  const cookieStore = cookies();
  
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        // Cookie handlers
      }
    }
  );
}
```

## Coding Standards

### TypeScript
- **Always use strict mode**
- Define interfaces for all component props
- Use type inference where possible, but be explicit when needed
- Avoid `any` type - use `unknown` or proper types
- Use enums for constants with multiple related values

### React/Next.js
- Prefer Server Components over Client Components
- Use Server Actions for mutations when possible
- Keep components small and focused (single responsibility)
- Extract reusable logic into custom hooks
- Use proper error boundaries
- Implement loading and error states

### Styling
- Use Tailwind utility classes
- Follow mobile-first responsive design
- Use the `cn()` utility for conditional classes
- Maintain consistent spacing (use Tailwind's spacing scale)
- Follow accessibility best practices (ARIA labels, semantic HTML)

### File Naming
- Components: PascalCase (e.g., `UserProfile.tsx`)
- Utilities: camelCase (e.g., `formatDate.ts`)
- Hooks: camelCase with 'use' prefix (e.g., `useAuth.ts`)
- Constants: UPPER_SNAKE_CASE (e.g., `API_ENDPOINTS.ts`)

### Testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustAGhosT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
