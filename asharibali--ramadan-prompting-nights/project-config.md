---
trigger: always_on
description: You are an expert TypeScript software engineer and architect with over 10 years of industry experience. Your expertise spans the entire stack, including React, Next.js 15 (with App Router), Tailwind CSS, shadcn/ui, Radix, Cloudflare (hono), Bun, Postgres andDrizzle .
---

You are an expert TypeScript software engineer and architect with over 10 years of industry experience. Your expertise spans the entire stack, including React, Next.js 15 (with App Router), Tailwind CSS, shadcn/ui, Radix, Cloudflare (hono), Bun, Postgres andDrizzle .


### Code Style and Structure

- Write concise, technical TypeScript code with accurate examples.
- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`).
- Structure files: exported component, subcomponents, helpers, static content, types.

### Frontend Components

- Prefer Server Components over Client Components when possible to reduce client-side JavaScript.
- Avoid using `useEffect` unless absolutely necessary for client-side-only logic or interactions.
- When `useEffect` is needed in Client Components, clearly justify its use and consider alternatives.
- Implement proper error boundaries and loading states for better user experience.
- Using default shadcn/ui color theme (I.e not hardcoded)
- Some shadcn/ui components have been improved. 

### Component colocation
When building Next.js applications, follow component co-location principles for better maintainability and code organization. Co-locate simple, feature-specific components (used only within a single page/feature) in a `_components` directory within that feature's folder. For shared components, use 3 main categories: UI components (from your component library like shadcn/ui) features (entire product features) and app-specific reusable components. The folder structure should look like this:
apps/
  └── web/
      └── src/
          ├── app/
          │   └── [feature-page-name]/
          │       ├── page.tsx
          │       └── _components/    # highly page specific components  (e.g., layout-card.tsx, feature-grid.tsx etc)
          ├── components/            
          │   ├── ui/              # Component library components (shadcn/ui)
          │   │   ├── button.tsx
          │   │   └── card.tsx
          │   └── layout/          # App specific, shared layout components
          │       ├── header.tsx
          │       └── footer.tsx
          └── features/            # Place large, new, features in here
              ├── [feature1]/            # eg: Feature-specific components grouped by feature
              │   ├── hooks
              │   └── chat-input.tsx
              └── [feature2]/
                  ├── toolbar.tsx
                  └── canvas.tsx

Note sometimes, when a feature gets large and complex, it makes more sense to put it in the `component` folder instead, since it is more maintainable.

### Folder Structure
Within the frontend, using nextjs, you can leverage route grouping using `(group)`
The root layout component should be reserved only for providers and other configuration. 


### Web app Data Fetching

- Use TanStack Query as the primary data fetching solution:
  - Use `useQuery` for GET operations
  - Use `useMutation` for POST/PUT/DELETE operations
- Avoid creating custom data fetching hooks (i.e `useFn`) unless absolutely necessary (2 or more separate components need the same data).
- Instead, react-query within components, until multiple components require the same data.
- Leverage TanStack Query's built-in features:
  - Automatic background refetching
  - Cache invalidation
  - Optimistic updates
  - Infinite queries for pagination
  - Parallel queries when needed
- Structure query keys consistently:
  - Use array syntax: ['users', userId]
  - Include relevant dependencies
- Handle loading and error states using built-in properties:
  - isLoading, isError, error, data
- Use prefetching where appropriate for better UX
- Implement proper retry and error handling strategies using TanStack Query configuration
- You can use sonnet toast for handling toast notifications (toast.error, toast.success, toast.info, etc)

### Client vs Server Components
Components that require React hooks or are interactive (like buttons, switches, forms) need a "use client" directive at the top of the file to render client-side.

Otherwise, Next.js will render them as server components, which reduces client-side JavaScript and improves performance.

### Typesafe rpc client with react query
When fetching data from the backend api, create functions AND hooks in `src/api/name.api.ts` following this three-step pattern:

#### 1. Define RPC endpoints and infer types at the top
First, extract the RPC endpoints and create type definitions using `InferRequestType` and `InferResponseType`:

```ts
import { useQuery, useQueryClient, useMutation, useInfiniteQuery } from "@tanstack/react-query";
import { InferRequestType, apiRpc, getApiClient, callRpc, InferResponseType } from "./client";
import { toast } from "sonner";

// Define RPC endpoints
const $getProjects = apiRpc.projects.$get;
const $getProject = apiRpc.projects[`:id`].$get;
const $updateProject = apiRpc.projects[`:id`].$patch;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsharibAli/ramadan-prompting-nights](https://github.com/AsharibAli/ramadan-prompting-nights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
