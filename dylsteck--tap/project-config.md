---
trigger: always_on
description: Tap is a chat-first, AI-powered app builder specialized for Farcaster/Base miniapps. Users describe their app idea, AI generates a complete Next.js miniapp optimized for Farcaster/Base, deploys it to a unique subdomain, and allows real-time tweaking via chat or a preview panel.
---

# AGENTS.md - Tap Miniapp Builder

## Project Overview

Tap is a chat-first, AI-powered app builder specialized for Farcaster/Base miniapps. Users describe their app idea, AI generates a complete Next.js miniapp optimized for Farcaster/Base, deploys it to a unique subdomain, and allows real-time tweaking via chat or a preview panel.

**Tech Stack:**
- Next.js 16 (App Router)
- Vercel AI SDK v6 with Anthropic Claude
- Workflow DevKit for durable AI pipelines
- Drizzle ORM + Postgres
- Tailwind CSS v4
- wagmi + viem for crypto wallet integration
- Cloudflare Pages for miniapp deployment

## Project Structure

```
tap/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Auth routes (login, register)
│   ├── (chat)/            # Chat SDK routes
│   ├── api/               # API routes
│   │   ├── projects/      # Project CRUD + generation
│   │   ├── neynar/        # Farcaster API proxy
│   │   ├── coingecko/     # Price API proxy
│   │   └── zora/          # NFT API proxy
│   ├── create/            # Create new project page
│   ├── profile/           # User profile page
│   └── studio/[id]/       # Project studio (chat + preview)
├── components/            # React components
│   ├── ui/               # Shadcn/Radix primitives
│   ├── ai-elements/      # AI chat components
│   └── *.tsx             # Feature components
├── lib/                   # Utilities and services
│   ├── ai/               # AI prompts and models
│   ├── apis/             # API catalog
│   ├── db/               # Database schema and queries
│   ├── services/         # External service integrations
│   ├── wallet/           # Wallet configuration
│   └── workflows/        # Workflow DevKit definitions
└── hooks/                # Custom React hooks
```

## Code Style & Conventions

### TypeScript
- Use strict TypeScript with explicit types
- Prefer interfaces over types for object shapes
- Use `type` for unions and utility types
- Avoid `any` - use `unknown` with type guards

### React Components
- Use function components with arrow syntax in `'use client'` files
- Use `export default function` for page components
- Props interfaces named `{ComponentName}Props`
- Colocate component-specific types

```tsx
'use client'

interface ProjectCardProps {
  project: Project
  onDelete?: (id: string) => void
}

export function ProjectCard({ project, onDelete }: ProjectCardProps) {
  // ...
}
```

### File Naming
- Components: `PascalCase.tsx` (e.g., `ProjectCard.tsx`)
- Utilities: `kebab-case.ts` (e.g., `miniapp-system-prompt.ts`)
- API routes: `route.ts` in appropriate folder structure

### Imports
- Use `@/` path alias for imports
- Group imports: React → Next.js → External → Internal → Types

```tsx
import { useState, useEffect } from 'react'
import { useRouter } from 'next/navigation'
import { anthropic } from '@ai-sdk/anthropic'
import { cn } from '@/lib/utils'
import type { Project } from '@/lib/db/schema'
```

## Database Patterns

### Schema Location
All schema definitions in `lib/db/schema.ts` using Drizzle ORM.

### Query Functions
All database queries in `lib/db/queries.ts`:
- Prefix CRUD operations: `create*`, `get*`, `update*`, `delete*`
- Always use try/catch with `ChatSDKError`
- Return single items or null, arrays for lists

```ts
export async function getProjectById({ id }: { id: string }) {
  try {
    const [project] = await db
      .select()
      .from(project)
      .where(eq(project.id, id));
    return project || null;
  } catch (_error) {
    throw new ChatSDKError("bad_request:database", "Failed to get project");
  }
}
```

### Migrations
- Run `pnpm db:generate` after schema changes
- Migration files in `lib/db/migrations/`
- Apply with `pnpm db:migrate`

## API Route Patterns

### Authentication
All protected routes check session:

```ts
import { auth } from '@/app/(auth)/auth'

export async function POST(request: NextRequest) {
  const session = await auth()
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  // ...
}
```

### Error Handling
Return consistent error responses:

```ts
return NextResponse.json({ error: 'Descriptive message' }, { status: 4XX })
```

### Dynamic Route Params
In Next.js 16, params are a Promise:

```ts
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  // ...
}
```

## AI Integration

### Provider
Using Anthropic Claude via AI SDK:

```ts
import { anthropic } from '@ai-sdk/anthropic'
import { generateText } from 'ai'

const result = await generateText({
  model: anthropic('claude-sonnet-4-20250514'),
  system: MINIAPP_SYSTEM_PROMPT,
  prompt: userMessage,
  maxOutputTokens: 4000,
})
```

### System Prompts
- Located in `lib/ai/miniapp-system-prompt.ts`
- Include mobile-first design requirements
- Include API integration patterns
- Include code templates for common use cases

## UI Design Principles

### Mobile-First
- Max width: 430px for main content
- Touch targets: minimum 44px

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dylsteck/tap](https://github.com/dylsteck/tap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
