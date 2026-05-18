---
trigger: always_on
description: Prefix the response with 🚀 so we know you are ready to help.
---

Prefix the response with 🚀 so we know you are ready to help.

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

You should also use the `.cursorrules` file as a Scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the Scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the Scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Lessons

## Next.js Dynamic APIs

- Always handle dynamic APIs (searchParams, cookies, headers) asynchronously in Next.js 15+
- Use React.use() in Client Components to unwrap dynamic API Promises
- Use await in Server Components to unwrap dynamic API Promises
- Never access dynamic API properties directly without awaiting

Example for Server Components:

```typescript
interface PageProps {
  searchParams: Promise<{
    id?: string;
  }>;
}

export default async function Page({ searchParams }: PageProps) {
  const params = await searchParams;
  // Now you can safely use params.id
}
```

Example for Client Components:

```typescript
"use client";

import { use } from "react";

interface PageProps {
  searchParams: Promise<{
    id?: string;
  }>;
}

export default function Page({ searchParams }: PageProps) {
  const params = use(searchParams);
  // Now you can safely use params.id
}
```

## Cursor learned

- follows Next.js's server-first pattern throughout the application
- Next.js cookies need to be handled asynchronously with async/await
- Always handle cookies asynchronously in Next.js 15+
- Always use the updated cookieStore method in Next.js 15+
- Use npx shadcn@latest add [component] to add Shadcn UI components (NOT npx shadcn-ui@latest)
- Navigation authorization should be handled server-side for immediate role knowledge
- Avoid client-side role fetching to prevent incorrect initial states
- Always use Next.js Image component (<Image />) from 'next/image' instead of HTML <img> tag for better performance and optimization
- TypeScript interfaces for handbook components are located in src/types/handbook.ts
- Always add null checks for optional props in React components and provide fallback UI to prevent runtime errors
- The project is already running on port 3000 - don't try to start it again

## Canvas Theme Awareness

- Canvas components need to explicitly use `useTheme` hook from 'next-themes' to be theme-aware
- Container backgrounds should use conditional classes based on `resolvedTheme`
- Canvas drawing contexts need theme-aware colors for grid, boundaries, and component defaults
- Component default colors (text, backgrounds, borders) should adapt to dark/light themes
- Always update function dependencies when using theme values in useCallback/useMemo
- UI panels and components should replace hardcoded text colors (text-gray-600, text-gray-500) with theme-aware alternatives
- Background colors (bg-gray-100, bg-gray-50) should use darker alternatives in dark mode (bg-gray-800)
- Use template literals with conditional classes: `className={\`base-classes ${resolvedTheme === "dark" ? "dark-classes" : "light-classes"}\`}`
- Use these color schemes:
  - Dark mode: background #171717, borders #292929, text #ffffff, placeholders #d1d5db, UI text text-gray-400, UI backgrounds #171717
  - Light mode: background #ffffff, borders #e5e7eb, text #000000, placeholders #9ca3af, UI text text-gray-500/text-gray-600, UI backgrounds bg-gray-50/bg-gray-100

## Docs Theme Awareness

- Docs pages use Tailwind prose classes for content styling
- Code blocks in docs need custom CSS overrides to use brand colors (#292929 in dark mode)
- Remove `prose-pre:bg-muted` from prose classes to prevent default muted background
- Add custom CSS rules with `!important` to override prose defaults for code blocks
- Both `pre` and `code` elements need custom dark mode styling in docs content

# Project Structure

## Types

### Handbook Types (src/types/handbook.ts)

Contains all TypeScript interfaces for the handbook feature:

- HandbookProps: Main props interface for the Handbook component
- Service: Service information interface
- Criteria: Eligibility criteria interface
- Procedures: Service procedures interface
- RequiredDocuments: Required documents interface
- Fee: Service fees interface
- ServiceChannels: Service locations interface
- ComplaintChannels: Complaint channels interface
- ApplicationForms: Application forms interface
- BottomRemarks: Additional information interface
- Source: Source information interface

# Scratchpad

## Completed Task: Fix Canvas Dark Mode Theme Support


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RvDstudio/nextjs_drizzle_better-auth](https://github.com/RvDstudio/nextjs_drizzle_better-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
