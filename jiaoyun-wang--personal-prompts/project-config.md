---
trigger: always_on
description: > Guidelines for AI coding agents working on this project.
---

# AGENTS.md

> Guidelines for AI coding agents working on this project.

## Project Overview

**prompts.chat** is a social platform for AI prompts built with Next.js 16. It allows users to share, discover, and collect prompts from the community. The project is open source and can be self-hosted with customizable branding, themes, and authentication.

### Tech Stack

- **Framework:** Next.js 16.0.7 (App Router) with React 19.2
- **Language:** TypeScript 5
- **Database:** PostgreSQL with Prisma ORM 6.19
- **Authentication:** NextAuth.js 5 (beta) with pluggable providers (credentials, GitHub, Google, Azure)
- **Styling:** Tailwind CSS 4 with Radix UI primitives
- **UI Components:** shadcn/ui pattern (components in `src/components/ui/`)
- **Internationalization:** next-intl with 11 supported locales
- **Icons:** Lucide React
- **Forms:** React Hook Form with Zod validation

## Project Structure

```
/
├── prisma/                 # Database schema and migrations
│   ├── schema.prisma       # Prisma schema definition
│   ├── migrations/         # Database migrations
│   └── seed.ts             # Database seeding script
├── public/                 # Static assets (logos, favicon)
├── messages/               # i18n translation files (en.json, es.json, etc.)
├── src/
│   ├── app/                # Next.js App Router pages
│   │   ├── (auth)/         # Auth pages (login, register)
│   │   ├── [username]/     # User profile pages
│   │   ├── admin/          # Admin dashboard
│   │   ├── api/            # API routes
│   │   ├── categories/     # Category pages
│   │   ├── prompts/        # Prompt CRUD pages
│   │   ├── feed/           # User feed
│   │   ├── discover/       # Discovery page
│   │   ├── settings/       # User settings
│   │   └── tags/           # Tag pages
│   ├── components/         # React components
│   │   ├── admin/          # Admin-specific components
│   │   ├── auth/           # Authentication components
│   │   ├── categories/     # Category components
│   │   ├── layout/         # Layout components (header, etc.)
│   │   ├── prompts/        # Prompt-related components
│   │   ├── providers/      # React context providers
│   │   ├── settings/       # Settings components
│   │   └── ui/             # shadcn/ui base components
│   ├── lib/                # Utility libraries
│   │   ├── ai/             # AI/OpenAI integration
│   │   ├── auth/           # NextAuth configuration
│   │   ├── config/         # Config type definitions
│   │   ├── i18n/           # Internationalization setup
│   │   ├── plugins/        # Plugin system (auth, storage)
│   │   ├── db.ts           # Prisma client instance
│   │   └── utils.ts        # Utility functions (cn)
│   └── i18n/               # i18n request handler
├── prompts.config.ts       # Main application configuration
├── prompts.csv             # Community prompts data source
└── package.json            # Dependencies and scripts
```

## Commands

```bash
# Development
npm run dev              # Start development server (localhost:3000)
npm run build            # Build for production (runs prisma generate first)
npm run start            # Start production server
npm run lint             # Run ESLint

# Database
npm run db:generate      # Generate Prisma client
npm run db:migrate       # Run database migrations
npm run db:push          # Push schema changes to database
npm run db:studio        # Open Prisma Studio
npm run db:seed          # Seed database with initial data

# Type checking
npx tsc --noEmit         # Check TypeScript types without emitting

# Translations
node scripts/check-translations.js  # Check for missing translations across locales
```

## Code Style Guidelines

### TypeScript

- Use TypeScript strict mode
- Prefer explicit types over `any`
- Use `interface` for object shapes, `type` for unions/intersections
- Functions: `camelCase` (e.g., `getUserData`, `handleSubmit`)
- Components: `PascalCase` (e.g., `PromptCard`, `AuthContent`)
- Constants: `UPPER_SNAKE_CASE` for true constants
- Files: `kebab-case.tsx` for components, `camelCase.ts` for utilities

### React/Next.js

- Use React Server Components by default
- Add `"use client"` directive only when client interactivity is needed
- Prefer server actions over API routes for mutations
- Use `next-intl` for all user-facing strings (never hardcode text)
- Import translations with `useTranslations()` or `getTranslations()`

### Component Pattern

```tsx
// Client component example
"use client";

import { useTranslations } from "next-intl";
import { Button } from "@/components/ui/button";

interface MyComponentProps {
  title: string;
  onAction: () => void;
}

export function MyComponent({ title, onAction }: MyComponentProps) {
  const t = useTranslations("namespace");
  
  return (
    <div className="space-y-4">
      <h2 className="text-lg font-semibold">{title}</h2>
      <Button onClick={onAction}>{t("actionLabel")}</Button>
    </div>
  );
}
```

### Styling

- Use Tailwind CSS utility classes
- Follow mobile-first responsive design (`sm:`, `md:`, `lg:` breakpoints)
- Use `cn()` utility from `@/lib/utils` for conditional classes
- Prefer Radix UI primitives via shadcn/ui components
- Keep component styling scoped and composable

### Database


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jiaoyun-Wang/personal-prompts](https://github.com/Jiaoyun-Wang/personal-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
