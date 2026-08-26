---
trigger: always_on
description: npm run dev          # Start development server (localhost:3000)
---

# AGENTS.md - Social Media App

## Build & Development Commands

```bash
# Development
npm run dev          # Start development server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server

# Code Quality
npm run lint         # Run ESLint

# Database
npx prisma generate  # Generate Prisma client
npx prisma db push   # Push schema to database
npx prisma studio    # Open Prisma Studio

# Environment
cp .env.example .env # Copy environment template (if exists)
```

## Project Overview

This is a Next.js 14 social media application with:
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Database**: PostgreSQL (Neon) with Prisma ORM
- **Auth**: Clerk
- **File Upload**: UploadThing
- **UI**: Tailwind CSS + shadcn/ui (Radix UI)

## Architecture

```
src/
├── app/                    # Next.js App Router (pages, layouts, API routes)
├── actions/                # Server Actions ("use server")
├── components/             # React components
│   ├── ui/                # shadcn/ui base components
│   └── *.tsx              # Feature components
├── lib/                    # Utilities and helpers
│   ├── prisma.ts          # Prisma singleton
│   ├── utils.ts           # cn() utility
│   ├── sanitize.ts        # XSS prevention
│   └── mentions.ts        # @mention extraction
├── middleware.ts           # Clerk middleware
└── types/                 # Shared TypeScript types
```

## Code Style Guidelines

### TypeScript

- Use explicit types for function parameters and return types
- Prefer `type` over `interface` for object shapes (see `src/types/index.ts`)
- Use `null` explicitly (not `undefined`) for optional fields
- Use optional chaining (`?.`) and nullish coalescing (`??`) appropriately

### React Components

**Server Components (default)**:
- No `"use client"` directive
- Fetch data directly using Server Actions or Prisma
- Props are serializable

**Client Components**:
- Add `"use client"` at the top of the file
- Use when: hooks, browser APIs, event handlers, interactive UI
- Keep client/server boundary clean - minimize client components

**Component Patterns**:
```tsx
// Client component with props interface
"use client";

import { cn } from "@/lib/utils";

type Props = {
  postId: string;
  hasLiked: boolean;
  canDelete?: boolean; // optional props last
};

export default function LikeButton({ postId, hasLiked, canDelete }: Props) {
  // component code
}
```

### Imports

**Order**:
1. React/core imports (`"use client"`, `React`)
2. Next.js imports (`next/navigation`, `next/image`)
3. Third-party libraries (`lucide-react`, `date-fns`)
4. Project imports (`@/components`, `@/actions`, `@/lib`)
5. Relative imports (`./`, `../`)

**Path aliases**: Use `@/` for src root (configured in tsconfig.json)

```tsx
import { useState } from "react";
import Image from "next/image";
import { HeartIcon } from "lucide-react";
import { cn } from "@/lib/utils";
import { likePost } from "@/actions/like.action";
import { Button } from "@/components/ui/button";
import "./styles.css";
```

### Server Actions

**File naming**: `*.action.ts`
**Directive**: `"use server"` at the top

```typescript
"use server";

import { prisma } from "@/lib/prisma";
import { getDbUserId } from "./user.action";
import { revalidatePath } from "next/cache";

export async function createPost(content: string, image: string) {
  try {
    const userId = await getDbUserId();
    if (!userId) return;

    const post = await prisma.post.create({
      data: { content, image, authorId: userId },
    });

    revalidatePath("/");
    return { success: true, post };
  } catch (error) {
    console.error("Failed to create post:", error);
    return { success: false, error: "Failed to create post" };
  }
}
```

**Error handling pattern**:
- Wrap in try/catch
- Log errors with `console.error`
- Return `{ success: false, error: "message" }` for user-facing errors
- Throw for critical errors that should halt execution

### Database (Prisma)

**Queries**: Use `select` to fetch only needed fields (avoid `include` when possible)

```typescript
const posts = await prisma.post.findMany({
  select: {
    id: true,
    content: true,
    author: {
      select: { id: true, username: true },
    },
  },
});
```

**Transactions**: Use `prisma.$transaction()` for atomic operations

```typescript
await prisma.$transaction([
  prisma.like.create({ data: { userId, postId } }),
  prisma.notification.create({ data: { ... } }),
]);
```

**Cascade deletes**: Use `onDelete: Cascade` in schema for related data cleanup

### UI Components (shadcn/ui)

**Button variants**:
```tsx
import { Button } from "@/components/ui/button";

// Variants: default, destructive, outline, secondary, ghost, link
<Button variant="destructive">Delete</Button>

// Sizes: default, sm, lg, icon
<Button size="sm">Small</Button>
```

**Utility function**: Use `cn()` from `@/lib/utils` for class merging

```tsx
import { cn } from "@/lib/utils";

<div className={cn("base-class", isActive && "active-class", className)} />
```

### Styling (Tailwind CSS)

- Use design system colors from `globals.css` (primary, secondary, destructive, etc.)
- Use `focus-visible` for keyboard accessibility
- Support dark mode using `dark:` prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Seb-fd/social-media-app](https://github.com/Seb-fd/social-media-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
