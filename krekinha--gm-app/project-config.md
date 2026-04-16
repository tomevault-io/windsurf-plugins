---
trigger: always_on
description: This rule provides comprehensive guidance for front-end development in Igniter.js + Next.js + Shadcn UI projects, ensuring consistent architecture, component patterns, and seamless integration with existing workflows.
---

# Front-End Development Workflow

This rule provides comprehensive guidance for front-end development in Igniter.js + Next.js + Shadcn UI projects, ensuring consistent architecture, component patterns, and seamless integration with existing workflows.

## 1. Project Architecture & File Organization

### 1.1 Mandatory Project Structure
**Presentation Layer Organization:**
```
src/features/[feature]/presentation/
├── components/           # UI components (client/server)
│   ├── [component].tsx
│   ├── [component].client.tsx    # Client-only components
│   └── [component].server.tsx    # Server-only components
├── hooks/               # React hooks (useXxx patterns)
│   ├── use-[feature].tsx
│   └── use-[feature]-form.tsx
├── contexts/            # React contexts for state management
│   └── [feature]-context.tsx
├── utils/              # Pure utility functions
│   └── [specific-util-for-feature].ts // 1 util per file
└── types/              # Local type definitions
    └── [feature]-types.ts
```

**Component File Naming Convention:**
```tsx
// ✅ CORRECT - Consistent naming
src/features/auth/presentation/
├── components/
│   ├── login-form.client.tsx      # Client component
│   ├── signup-form.client.tsx     # Client component
│   └── user-avatar.server.tsx     # Server component
├── hooks/
│   ├── use-auth-context.tsx       # Context hook
│   └── use-auth-form.tsx          # Form hook
└── contexts/
    └── auth-context.tsx           # React context
```

### 1.2 Component Architecture Patterns

**Server vs Client Components:**
```tsx
// ✅ CORRECT - Server component (default)
export default function BoardList({ boards }: { boards: Board[] }) {
  return (
    <div>
      {boards.map(board => (
        <BoardCard key={board.id} board={board} />
      ))}
    </div>
  )
}

// ✅ CORRECT - Client component (interactive)
'use client'

export function BoardCard({ board }: { board: Board }) {
  const [isEditing, setIsEditing] = useState(false)

  return (
    <Card onClick={() => setIsEditing(true)}>
      {/* Interactive content */}
    </Card>
  )
}
```

**Component Composition Pattern:**
```tsx
// ✅ CORRECT - Compound component pattern
function BoardActions({ children }: { children: ReactNode }) {
  return (
    <div className="flex gap-2">
      {children}
    </div>
  )
}

function BoardActionsEdit({ boardId }: { boardId: string }) {
  return <Button>Edit</Button>
}

function BoardActionsDelete({ boardId }: { boardId: string }) {
  return <Button variant="destructive">Delete</Button>
}

// Usage
<BoardActions>
  <BoardActionsEdit boardId={board.id} />
  <BoardActionsDelete boardId={board.id} />
</BoardActions>
```

## 2. Igniter.js Client Integration Protocol

**⚠️ IMPORTANT NOTE:** All examples in this section have been corrected based on the actual implementation found in the project codebase. The Igniter.js client has hooks available:

- **Queries**: `api.endpoint.useQuery()` returns `{ data: { data: T }, isLoading, error }`
- **Mutations**: `api.endpoint.useMutation()` returns hook with `{ mutate, data, isLoading, isSuccess, isError, error }`
- **Invalidation**: `useIgniterQueryClient()` provides `invalidate()` method with string-based query keys
- **Best Practice**: Use the provided hooks for consistent state management and automatic loading states

### 2.1 Client Usage Patterns

**Server-Side Data Fetching:**
```tsx
// ✅ CORRECT - Server component data fetching
export default async function BoardsPage() {
  const boards = await api.boards.list.query()

  return (
    <BoardsGrid boards={boards.data || []} />
  )
}

// ✅ CORRECT - Server action data mutation
'use server'

export async function createBoardAction(formData: FormData) {
  const boardData = {
    title: formData.get('title') as string,
    description: formData.get('description') as string,
  }

  const result = await api.boards.create.mutate({ body: boardData })

  if (result.data) {
    revalidatePath('/app/boards')
    redirect(`/app/boards/${result.data.id}`)
  }
}
```

**Client-Side Interactive Patterns:**
```tsx
// ✅ CORRECT - Client component with Igniter.js hooks
'use client'

import { api } from '@/igniter.client'

function BoardsList() {
  const { data: boardsData, isLoading, error } = api.boards.list.useQuery()

  const boards = boardsData?.data // Access nested response structure

  if (isLoading) return <BoardsSkeleton />
  if (error) return <BoardsError error={error} />

  return (
    <div className="grid gap-4">
      {boards?.map(board => (
        <BoardCard key={board.id} board={board} />
      ))}
    </div>
  )
}

// ✅ CORRECT - Using Igniter.js useMutation hook with proper callbacks
const createBoardMutation = api.boards.create.useMutation({
  onSuccess: (result: any) => {
    console.log('Board created:', result?.data)

    // Handle success - invalidate queries and navigate
    queryClient.invalidate('boards.list') // from igniter.client.ts
    router.push(`/app/boards/${result?.data?.id}`)
  },
  onError: (error: any) => {
    console.error('Failed to create board:', error)
    // Handle error - could show toast notification
  }
})


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Krekinha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
