---
trigger: always_on
description: This file contains essential context about the project structure, technologies, and conventions to help Claude understand and work effectively within this codebase.
---

# CLAUDE.md - CodeGuide Starter Kit

This file contains essential context about the project structure, technologies, and conventions to help Claude understand and work effectively within this codebase.

## Project Overview

**CodeGuide Starter Kit** is a modern Next.js starter template featuring authentication, database integration, AI capabilities, and a comprehensive UI component system.

### Core Technologies

- **Framework**: Next.js 15 with App Router (`/src/app` directory structure)
- **Language**: TypeScript with strict mode enabled
- **Styling**: TailwindCSS v4 with CSS custom properties
- **UI Components**: shadcn/ui (New York style) with Lucide icons
- **Authentication**: Clerk with middleware protection
- **Database**: Supabase with third-party auth integration
- **AI Integration**: Vercel AI SDK with support for Anthropic Claude and OpenAI
- **Theme System**: next-themes with dark mode support

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes
│   │   └── chat/          # AI chat endpoint
│   ├── globals.css        # Global styles with dark mode
│   ├── layout.tsx         # Root layout with providers
│   └── page.tsx           # Home page with status dashboard
├── components/
│   ├── ui/                # shadcn/ui components (40+ components)
│   ├── chat.tsx           # AI chat interface
│   ├── setup-guide.tsx    # Configuration guide
│   ├── theme-provider.tsx # Theme context provider
│   └── theme-toggle.tsx   # Dark mode toggle components
├── lib/
│   ├── utils.ts           # Utility functions (cn, etc.)
│   ├── supabase.ts        # Supabase client configurations
│   ├── user.ts            # User utilities using Clerk
│   └── env-check.ts       # Environment validation
└── middleware.ts          # Clerk authentication middleware
```

## Key Configuration Files

- **package.json**: Dependencies and scripts
- **components.json**: shadcn/ui configuration (New York style, neutral colors)
- **tsconfig.json**: TypeScript configuration with path aliases (`@/`)
- **.env.example**: Environment variables template
- **SUPABASE_CLERK_SETUP.md**: Integration setup guide

## Authentication & Database

### Clerk Integration
- Middleware protects `/dashboard(.*)` and `/profile(.*)` routes
- Components: `SignInButton`, `SignedIn`, `SignedOut`, `UserButton`
- User utilities in `src/lib/user.ts` use `currentUser()` from Clerk

### Supabase Integration
- **Client**: `createSupabaseServerClient()` for server-side with Clerk tokens  
- **RLS**: Row Level Security uses `auth.jwt() ->> 'sub'` for Clerk user IDs
- **Example Migration**: `supabase/migrations/001_example_tables_with_rls.sql`

#### Supabase Client Usage Patterns

**Server-side (Recommended for data fetching):**
```typescript
import { createSupabaseServerClient } from "@/lib/supabase"

export async function getServerData() {
  const supabase = await createSupabaseServerClient()
  
  const { data, error } = await supabase
    .from('posts')
    .select('*')
    .order('created_at', { ascending: false })
  
  if (error) {
    console.error('Database error:', error)
    return null
  }
  
  return data
}
```

**Client-side (For interactive operations):**
```typescript
"use client"

import { supabase } from "@/lib/supabase"
import { useAuth } from "@clerk/nextjs"

function ClientComponent() {
  const { getToken } = useAuth()

  const fetchData = async () => {
    const token = await getToken()
    
    // Pass token manually for client-side operations
    const { data, error } = await supabase
      .from('posts')
      .select('*')
      .auth(token)
    
    return data
  }
}
```

## UI & Styling

### TailwindCSS Setup
- **Version**: TailwindCSS v4 with PostCSS
- **Custom Properties**: CSS variables for theming
- **Dark Mode**: Class-based with `next-themes`
- **Animations**: `tw-animate-css` package included

### shadcn/ui Components
- **Style**: New York variant
- **Theme**: Neutral base color with CSS variables
- **Icons**: Lucide React
- **Components Available**: 40+ UI components (Button, Card, Dialog, etc.)

### Theme System
- **Provider**: `ThemeProvider` in layout with system detection
- **Toggle Components**: `ThemeToggle` (dropdown) and `SimpleThemeToggle` (button)
- **Persistence**: Automatic theme persistence across sessions

## AI Integration

### Vercel AI SDK
- **Endpoint**: `/api/chat/route.ts`
- **Providers**: Anthropic Claude and OpenAI support
- **Chat Component**: Real-time streaming chat interface
- **Authentication**: Requires Clerk authentication

## Development Conventions

### File Organization
- **Components**: Use PascalCase, place in appropriate directories
- **Utilities**: Place reusable functions in `src/lib/`
- **Types**: Define alongside components or in dedicated files
- **API Routes**: Follow Next.js App Router conventions

### Import Patterns
```typescript
// Path aliases (configured in tsconfig.json)
import { Button } from "@/components/ui/button"
import { getCurrentUser } from "@/lib/user"
import { supabase } from "@/lib/supabase"

// External libraries
import { useTheme } from "next-themes"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CodeGuide-dev/codeguide-starter-lite](https://github.com/CodeGuide-dev/codeguide-starter-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
