---
trigger: always_on
description: This application is a unified chatbot platform that allows users to chat with AI models from
---

This application is a unified chatbot platform that allows users to chat with AI models from
any of the configured providers.

## Architecture

The application is built with the following stack:
- [Catalyst](https://github.com/kovrichard/catalyst) for the core framework. This includes:
    * Bun.js
    * Prisma
    * Next.js
    * Tailwind CSS
    * Shadcn UI
    * tRPC
    * Husky
    * Biome
    * Better Auth
    * Zod
    * Winston
- [Vercel AI SDK](https://ai-sdk.dev/docs/introduction) for accessing the AI models.

Folder structure:

```bash
src/                      # Source root
  app/                    # Next.js app directory
  app/api/chat/route.ts   # Main chat API endpoint (streams AI responses)
  components/             # Next.js components
  components/ui/          # Shadcn UI components
  components/ai-elements/ # AI elements components
  hooks/                  # React hooks
  lib/                    # Library functions
  lib/queries/            # Client-side React Query hooks (optimistic updates, cache management)
  lib/actions/            # Next.js server actions (called from mutations)
  lib/dao/                # Database access layer (Prisma queries, called from tRPC and actions)
  lib/trpc/routers/       # tRPC route definitions (called from React Query hooks)
  lib/contexts/           # React contexts
  stores/                 # Zustand stores (client-side reactive state)
  types/                  # TypeScript types
```

### Data flow layers

Client-side data flows through these layers (top to bottom):

1. **Zustand stores** (`stores/`) — transient client state (selected model, UI flags)
2. **React Query hooks** (`lib/queries/`) — cached server state with optimistic updates
3. **tRPC routes** (`lib/trpc/routers/`) — server endpoints for reads
4. **Server actions** (`lib/actions/`) — server endpoints for writes
5. **DAOs** (`lib/dao/`) — Prisma database queries

**CRITICAL RULE - Data Fetching:**
- React Query hooks (`lib/queries/`) MUST fetch data through **tRPC only** - never import DAO functions directly
- Use `useTRPC()` hook and call tRPC procedures (e.g., `trpc.projects.queryOptions()`)
- DAO functions are for tRPC routes and server actions only
- Server actions (`lib/actions/`) can call DAO functions directly for writes
- Violating this breaks the application as DAO functions are `server-only` and cannot be imported directly in client-side code.

### Client-side caching

React Query cache is persisted to **IndexedDB** (key: `"fyzz-chat-query-cache"`) via
`lib/trpc/query-persister.ts`, so data survives page reloads. Configuration is in
`lib/trpc/query-client.ts` (`staleTime: 15s`, `refetchOnMount: false`). Queries opt out of
persistence with `meta: { persist: false }`.

### Conversation lifecycle

- **New conversation:** Landing page stores initial message/model in `InitialMessageContext`
  (transient), navigates to `/chat/[id]`. `message-list.tsx` creates the conversation
  optimistically in React Query cache, then `handleSubmit` hits the chat API which creates
  the DB row via `getOrCreateConversation`.
- **Existing conversation:** `useConversation(id)` loads from IndexedDB cache or tRPC.
  Model is synced to Zustand store via a `useEffect` in `message-list.tsx`.

### Server-side data loading

When a page needs multiple server-side queries, do NOT `Promise.all` them and block rendering
until all resolve. Instead:

1. `await` only the fast/critical query directly (e.g., lightweight metadata).
2. Wrap heavier queries in inline async server components, each inside a `<Suspense>` boundary
   with a skeleton fallback.
3. Create skeleton components (`Skeleton` from `components/ui/skeleton`) that match the real
   content's dimensions to avoid layout shifts when data streams in.

This lets the shell render instantly while expensive data streams in progressively.

### Boilerplate-first rendering

For any page with data-bound widgets, default to **boilerplate-first**: render the static
structure synchronously in the page server component, and wrap only data-bound widgets in
`<Suspense>` islands. Extends the rules above with five concrete habits:

1. **Hoist static layout into the page.** Headings, descriptions, container chrome, form
   labels, navigation links, "back" buttons — all of this can render before any data
   resolves. Pull it out of feature components and write it directly in the page so a
   reader can see at a glance what costs data and what's free.
2. **One Suspense per data slice, not per feature.** A page with a model select, a toggle,
   two name inputs, and a memory list gets four Suspense islands — not one wrapping the
   whole form. Each island uses the React-cached DAO (`React.cache(...)`) so multiple
   loaders for the same query still hit the DB once per request.
3. **Match the full rendered box, not just the height class.** The rule is: when the
   skeleton swaps for the real widget, **nothing on the page should move**. That means
   matching everything that contributes to layout — height (including border + padding +
   content line-height, not just the upstream Tailwind class), width, border-radius, and
   the surrounding spacing (gap, margin, parent grid/flex). Don't trust shadcn defaults

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fyzz-Chat/fyzz-chat](https://github.com/Fyzz-Chat/fyzz-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
