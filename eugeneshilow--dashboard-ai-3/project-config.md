---
trigger: always_on
description: Use the project specification and guidelines as you build the project AI Dashboard
---

# Project Instructions

Use the project specification and guidelines as you build the project AI Dashboard

Write the complete code for every step. Do not get lazy. Be sure to input all project data when answering the queries.

Your goal is to completely finish whatever the user asks for.

## Overview

This is an AI Dashboard for the AI market. It shows the market size growth and the latest news in the AI market.

## Tech Stack

- Frontend: Next.js, Tailwind, Shadcn, Framer Motion
- Backend: Postgres, Supabase, Drizzle ORM, Server Actions
- Auth: Clerk
- Payments: Robokassa
- Deployment: Vercel

## Project Structure

### General Structure

- `actions` - Server actions
  - `db` - Database related actions
  - Other actions
- `app` - Next.js app router
  - `api` - API routes
  - `route` - An example route (each of these are only if needed)
    - `_components` - One-off components for the route
    - `layout.tsx` - Layout for the route
    - `page.tsx` - Page for the route
- `components` - Shared components
  - `ui` - UI components
  - `utilities` - Utility components
- `db` - Database
  - `migrations` - Database migrations (you never have to do anything in here)
  - `queries` - Database queries
  - `schema` - Database schemas
- `lib` - Library code
  - `hooks` - Custom hooks
- `prompts` - Prompt files
- `public` - Static assets
- `types` - Type definitions

### Project Specific Structure

## Rules

Follow these rules when building the project.

### General Rules

- Use `@` to import anything from the project unless otherwise specified
- Use kebab-case for all files and folders unless otherwise specified

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
export type ActionState<T> = {
  isSuccess: boolean
  message: string
  data?: T
}
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

#### Components

- Use divs instead of other html tags unless otherwise specified
- Separate the main parts of a component's html with an extra blank line for visual spacing
- Use actions, not queries, in the app
- Always tag a component with either `use server` or `use client` at the top, including layouts and pages
- Add props where needed when defining new components

Example of a component with props:

```tsx
interface SearchInputProps {
  onSearch: (query: string) => void
  className?: string
}
```

##### Organization

- All components be named using kebab case like `example-component.tsx` unless otherwise specified
- Put components in `/_components` in the route if one-off components
- Put components in `/components` from the root if shared components

##### Data Fetching

- Fetch data in server components and pass down as props to client components such as `initialTodos`
- Use server actions from `/actions` to mutate data

##### Server Components

- Use `"use server"` at the top of the file
- Implement Suspense for asynchronous data fetching
- Use a separate fetcher component for data loading (see example below)
- Make sure to use `async` for all pages and components if they are server components

Example of a server page:

```tsx
"use server"

import { Suspense } from "react"
import { SomeAction } from "@/actions/some-actions"
import SomeComponent from "./_components/some-component"
import SomeSkeleton from "./_components/some-skeleton"

export default async function ExampleServerPage({
  params
}: {
  params: { id: string }
}) {
  return (
    <Suspense fallback={<SomeSkeleton className="some-class" />}>
      <SomeComponentFetcher id={params.id} />
    </Suspense>
  )
}

async function SomeComponentFetcher({ id }: { id: string }) {
  const { data } = await SomeAction(id)

  return (
    <SomeComponent className="some-class" initialData={data || []} id={id} />
  )
}
```

Example of a server component:

```tsx
"use server"

interface ExampleServerComponentProps {
  // Your props here
}

export async function ExampleServerComponent({
  props
}: ExampleServerComponentProps) {
  // Your code here
}
```

##### Client Components

- Use `"use client"` at the top of the file

Example of a client page:

```tsx
"use client"

export default function ExampleClientPage() {
  // Your code here
}
```

Example of a client component:

```tsx
"use client"

interface ExampleClientComponentProps {
  // Your props here
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eugeneshilow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
