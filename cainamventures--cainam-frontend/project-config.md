---
trigger: always_on
description: Use specification and guidelines as you build the app.
---

# Project Instructions

Use specification and guidelines as you build the app.

Write the complete code for every step. Do not get lazy.

Your goal is to completely finish whatever I ask for.

You will see <ai_context> tags in the code. These are context tags that you should use to help you understand the codebase.

## Overview

This is a web app template.

## Tech Stack

- Frontend: Next.js, Tailwind, Shadcn, Framer Motion
- Backend: Postgres, Supabase, Drizzle ORM, Server Actions
- Auth: Clerk
- Payments: Stripe
- Analytics: PostHog
- Deployment: Vercel

## Project Structure

- `actions` - Server actions
  - `db` - Database related actions
  - Other actions
- `app` - Next.js app router
  - `api` - API routes
  - `route` - An example route
    - `_components` - One-off components for the route
    - `layout.tsx` - Layout for the route
    - `page.tsx` - Page for the route
- `components` - Shared components
  - `ui` - UI components
  - `utilities` - Utility components
- `db` - Database
  - `schema` - Database schemas
- `lib` - Library code
  - `hooks` - Custom hooks
- `prompts` - Prompt files
- `public` - Static assets
- `types` - Type definitions

## Rules

Follow these rules when building the app.

### General Rules

- Use `@` to import anything from the app unless otherwise specified
- Use kebab case for all files and folders unless otherwise specified
- Don't update shadcn components unless otherwise specified

#### Env Rules

- If you update environment variables, update the `.env.example` file
- All environment variables should go in `.env.local`
- Do not expose environment variables to the frontend
- Use `NEXT_PUBLIC_` prefix for environment variables that need to be accessed from the frontend
- You may import environment variables in server actions and components by using `process.env.VARIABLE_NAME`

#### Type Rules

Follow these rules when working with types.

- When importing types, use `@/types`
- Name files like `example-types.ts`
- All types should go in `types`
- Make sure to export the types in `types/index.ts`
- Prefer interfaces over type aliases
- If referring to db types, use `@/db/schema` such as `SelectTodo` from `todos-schema.ts`

An example of a type:

`types/actions-types.ts`

```ts
export type ActionState<T> =
  | { isSuccess: true; message: string; data: T }
  | { isSuccess: false; message: string; data?: never }
```

And exporting it:

`types/index.ts`

```ts
export * from "./actions-types"
```

### Frontend Rules

Follow these rules when working on the frontend.

It uses Next.js, Tailwind, Shadcn, and Framer Motion.

#### General Rules

- Use `lucide-react` for icons
- useSidebar must be used within a SidebarProvider

#### Components

- Use divs instead of other html tags unless otherwise specified
- Separate the main parts of a component's html with an extra blank line for visual spacing
- Always tag a component with either `use server` or `use client` at the top, including layouts and pages

##### Organization

- All components be named using kebab case like `example-component.tsx` unless otherwise specified
- Put components in `/_components` in the route if one-off components
- Put components in `/components` from the root if shared components

##### Data Fetching

- Fetch data in server components and pass the data down as props to client components.
- Use server actions from `/actions` to mutate data.

##### Server Components

- Use `"use server"` at the top of the file.
- Implement Suspense for asynchronous data fetching to show loading states while data is being fetched.
- If no asynchronous logic is required for a given server component, you do not need to wrap the component in `<Suspense>`. You can simply return the final UI directly since there is no async boundary needed.
- If asynchronous fetching is required, you can use a `<Suspense>` boundary and a fallback to indicate a loading state while data is loading.
- Server components cannot be imported into client components. If you want to use a server component in a client component, you must pass the as props using the "children" prop
- params in server pages should be awaited such as `const { courseId } = await params` where the type is `params: Promise<{ courseId: string }>`

Example of a server layout:

```tsx
"use server"

export default async function ExampleServerLayout({
  children
}: {
  children: React.ReactNode
}) {
  return children
}
```

Example of a server page (with async logic):

```tsx
"use server"

import { Suspense } from "react"
import { SomeAction } from "@/actions/some-actions"
import SomeComponent from "./_components/some-component"
import SomeSkeleton from "./_components/some-skeleton"

export default async function ExampleServerPage() {
  return (
    <Suspense fallback={<SomeSkeleton className="some-class" />}>
      <SomeComponentFetcher />
    </Suspense>
  )
}

async function SomeComponentFetcher() {
  const { data } = await SomeAction()
  return <SomeComponent className="some-class" initialData={data || []} />
}
```

Example of a server page (no async logic required):

```tsx
"use server"

import SomeClientComponent from "./_components/some-client-component"

// In this case, no asynchronous work is being done, so no Suspense or fallback is required.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CainamVentures) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
