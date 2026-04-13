---
trigger: always_on
description: This is the Project Genesis sandbox - a development environment for building SaaS applications using AI-assisted development with Linear task management.
---

# Project Genesis - Cursor AI Rules

# Last Updated: October 3, 2025

# Project: Sandbox (Development Environment)

# Owner: Farid Kheloco

## Project Context

This is the Project Genesis sandbox - a development environment for building SaaS applications using AI-assisted development with Linear task management.

Repository: https://github.com/purelyapp/projects
Vercel Team: Purely Startup (team_8lsIg2OsJ2ORppOIGFUZt55M)
Supabase Project: sandbox (grqlsdaapzqrsbaepiuj)

## Tech Stack

- **Framework**: Next.js 15.5.4 (App Router)
- **Language**: TypeScript 5 (strict mode)
- **Styling**: Tailwind CSS 4
- **UI Components**: Shadcn/ui (to be added)
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth
- **Deployment**: Vercel
- **State Management**: React hooks (useState, useContext)
- **Data Fetching**: React Server Components where possible

## Code Standards

### TypeScript Rules

- **ALWAYS** use TypeScript strict mode
- **NEVER** use `any` - use `unknown` if type is truly dynamic
- Define interfaces for all props and function parameters
- Use type inference when obvious
- Export types alongside components

Example:

```typescript
// ✅ Good
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

export function Button({
  label,
  onClick,
  variant = "primary",
  disabled = false,
}: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled} className={`btn-${variant}`}>
      {label}
    </button>
  );
}

// ❌ Bad
export function Button(props: any) {
  return <button>{props.label}</button>;
}
```

### React Component Standards

- Use functional components only (no class components)
- Use named exports (not default exports)
- Keep components small and focused (< 200 lines)
- Extract complex logic into custom hooks
- Use Server Components by default (add 'use client' only when needed)
- One component per file

Component structure:

```typescript
"use client"; // Only if you need client-side features

import { useState } from "react"; // Imports first
// Types second
interface ComponentProps {
  title: string;
  onSave?: () => void;
}

// Component third
export function ComponentName({ title, onSave }: ComponentProps) {
  // 1. Hooks
  const [state, setState] = useState<string>("");

  // 2. Effects
  useEffect(() => {
    // effect logic
  }, []);

  // 3. Handlers
  const handleClick = () => {
    setState("clicked");
    onSave?.();
  };

  // 4. Early returns
  if (!title) return null;

  // 5. Render
  return (
    <div>
      <h1>{title}</h1>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

### File Naming Conventions

- **Components**: `PascalCase.tsx` (e.g., `UserProfile.tsx`, `LoginForm.tsx`)
- **Utilities**: `camelCase.ts` (e.g., `formatDate.ts`, `validateEmail.ts`)
- **API routes**: `route.ts` (Next.js App Router convention)
- **Types**: `types.ts` or `ComponentName.types.ts`
- **Hooks**: `useCamelCase.ts` (e.g., `useAuth.ts`, `useLocalStorage.ts`)

### Directory Structure

```
app/
  ├── (auth)/              # Auth-related pages (route groups)
  │   ├── login/
  │   └── signup/
  ├── (dashboard)/         # Protected pages
  │   ├── profile/
  │   └── settings/
  ├── api/                 # API routes
  │   ├── auth/
  │   └── users/
  ├── layout.tsx           # Root layout
  └── page.tsx             # Home page
components/
  ├── ui/                  # Shadcn components
  ├── forms/               # Form components
  ├── layouts/             # Layout components
  └── ...                  # Feature components
lib/
  ├── supabase.ts          # Supabase client
  ├── utils.ts             # Utility functions
  ├── constants.ts         # App constants
  └── ...                  # Other libraries
hooks/
  ├── useAuth.ts           # Authentication hook
  └── ...                  # Custom hooks
types/
  ├── database.ts          # Database types
  └── ...                  # Type definitions
```

### API Route Standards

- **Always** validate input with Zod
- **Always** handle errors gracefully
- Return consistent JSON response format
- Use appropriate HTTP status codes
- Include error messages that are helpful but not revealing

API Route Template:

```typescript
import { NextResponse } from "next/server";
import { z } from "zod";
import { supabase } from "@/lib/supabase";

// Define request schema
const createUserSchema = z.object({
  email: z.string().email("Invalid email address"),
  full_name: z.string().min(1, "Name is required").max(100),
  avatar_url: z.string().url().optional(),
});

export async function POST(req: Request) {
  try {
    // 1. Parse and validate request body
    const body = await req.json();
    const validatedData = createUserSchema.parse(body);

    // 2. Business logic
    const { data, error } = await supabase
      .from("profiles")
      .insert([validatedData])
      .select()
      .single();

    if (error) {
      throw new Error(`Database error: ${error.message}`);
    }

    // 3. Return success response
    return NextResponse.json(
      { data, message: "User created successfully" },
      { status: 201 }
    );
  } catch (error) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/purelyworks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
