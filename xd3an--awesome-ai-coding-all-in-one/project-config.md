---
trigger: always_on
description: Cursor rules for Next.js App Router with TanStack Query v5, covering the HydrationBoundary pattern, Server Actions as mutations, and optimistic updates.
---

You are an expert in Next.js (App Router), TanStack Query v5, TypeScript, and combining server components with client-side data fetching.

# Next.js App Router + TanStack Query v5 Guidelines

## Architecture Philosophy
- Server Components fetch data directly (no TanStack Query needed there)
- TanStack Query lives in Client Components for interactive, real-time, or user-triggered data
- Use React Server Components for initial page data; TanStack Query for mutations, polling, and optimistic updates
- Hydrate the Query cache from server to avoid client waterfalls on first load

## Provider Setup with Hydration
```tsx
// src/providers/query-provider.tsx
'use client'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { useState } from 'react'

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000,
            retry: (count, error: any) => error?.status !== 404 && count < 2,
          },
        },
      }),
  )

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}

// src/app/layout.tsx
import { QueryProvider } from '@/providers/query-provider'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <QueryProvider>{children}</QueryProvider>
      </body>
    </html>
  )
}
```

## Hydration Pattern (Server → Client Cache)
- Prefetch in Server Components, dehydrate state, rehydrate in client
- This eliminates client-side loading states on first render
```tsx
// src/app/posts/page.tsx (Server Component)
import { dehydrate, HydrationBoundary, QueryClient } from '@tanstack/react-query'
import { postsQueryOptions } from '@/queries/posts'
import { PostsList } from './_components/posts-list'

export default async function PostsPage() {
  const queryClient = new QueryClient()
  await queryClient.prefetchQuery(postsQueryOptions())

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <PostsList />
    </HydrationBoundary>
  )
}

// src/app/posts/_components/posts-list.tsx
'use client'
import { useQuery } from '@tanstack/react-query'
import { postsQueryOptions } from '@/queries/posts'

export function PostsList() {
  // Reads from pre-populated cache — no loading spinner
  const { data: posts } = useQuery(postsQueryOptions())
  return <ul>{posts?.map(p => <li key={p.id}>{p.title}</li>)}</ul>
}
```

## Query Definitions
```ts
// src/queries/posts.ts
import { queryOptions } from '@tanstack/react-query'

export const postKeys = {
  all: ['posts'] as const,
  lists: () => [...postKeys.all, 'list'] as const,
  list: (filters?: PostFilters) => [...postKeys.lists(), { filters }] as const,
  details: () => [...postKeys.all, 'detail'] as const,
  detail: (id: string) => [...postKeys.details(), id] as const,
}

export const postsQueryOptions = (filters?: PostFilters) =>
  queryOptions({
    queryKey: postKeys.list(filters),
    queryFn: () => fetch(`/api/posts`).then(r => r.json()),
  })

export const postDetailQueryOptions = (id: string) =>
  queryOptions({
    queryKey: postKeys.detail(id),
    queryFn: () => fetch(`/api/posts/${id}`).then(r => r.json()),
    staleTime: 1000 * 60 * 5,
  })
```

## Server Actions + Mutations
- Use Next.js Server Actions as the `mutationFn` in TanStack Query mutations
- This gives you type-safe server mutations WITH optimistic update/rollback capabilities
```tsx
// src/app/posts/actions.ts
'use server'
import { revalidatePath } from 'next/cache'

export async function createPost(data: { title: string; body: string }) {
  const post = await db.post.create({ data })
  revalidatePath('/posts')
  return post
}

// src/app/posts/_components/create-post-form.tsx
'use client'
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { createPost } from '../actions'
import { postKeys } from '@/queries/posts'

export function CreatePostForm() {
  const queryClient = useQueryClient()
  const mutation = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: postKeys.lists() })
    },
  })

  return (
    <button
      onClick={() => mutation.mutate({ title: 'New Post', body: '...' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Creating...' : 'Create Post'}
    </button>
  )
}
```

## Optimistic Updates with Server Actions
```tsx
const mutation = useMutation({
  mutationFn: updatePost,
  onMutate: async (updated) => {
    await queryClient.cancelQueries({ queryKey: postKeys.detail(updated.id) })
    const previous = queryClient.getQueryData(postKeys.detail(updated.id))
    queryClient.setQueryData(postKeys.detail(updated.id), (old: Post) => ({ ...old, ...updated }))
    return { previous }
  },
  onError: (_, updated, ctx) => {
    queryClient.setQueryData(postKeys.detail(updated.id), ctx?.previous)
  },
  onSettled: (_, __, updated) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
