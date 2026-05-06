---
trigger: always_on
description: You are an AI pair programmer working on the MCP App Demo project. Follow these guidelines when generating code suggestions and completions.
---

# GitHub Copilot Instructions for MCP App Demo

You are an AI pair programmer working on the MCP App Demo project. Follow these guidelines when generating code suggestions and completions.

## Technology Stack & General Guidelines

This project uses:

- **TypeScript** (strict mode enabled)
- **Vite** for build tooling
- **TanStack Start** for routing and SSR
- **Tailwind CSS** for styling
- **Shadcn/ui** for UI components
- **Zod** for validation
- **React Query (TanStack Query)** for server state

### Code Quality Standards

- **Never use `any` type** - always use proper TypeScript types or let TypeScript infer
- **Write self-documenting code** with clear variable and function names
- **Add comments only for complex business logic** that isn't obvious from the code
- **Follow existing linting and Prettier rules** - the project uses ESLint and Prettier
- **Suggest running `npm run lint:fix`** after significant code changes

## TypeScript Patterns

```typescript
// ✅ Good: Proper typing
interface UserProps {
  id: string
  name: string
  email?: string
}

// ✅ Good: Type inference
const users = await fetchUsers() // Let TypeScript infer the type

// ❌ Bad: Using any
const data: any = await fetchData()
```

## React Component Patterns

### Prefer Function Components with TypeScript

```typescript
// ✅ Good: Function component with proper typing
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

export default function Button({ children, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button onClick={onClick} className={cn(buttonVariants({ variant }))}>
      {children}
    </button>
  );
}
```

### State Management Patterns

```typescript
// ✅ Good: Local state with useState
const [isOpen, setIsOpen] = useState(false)

// ✅ Good: Complex state with useReducer
const [state, dispatch] = useReducer(modalReducer, initialState)

// ✅ Good: Context for global state
const { user, setUser } = useContext(UserContext)

// ✅ Good: React Query for client-side server state (when route loaders aren't suitable)
const { data: users, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
})
```

### React Query vs Route Loaders

**Prefer Route Loaders for:**

- Initial page data that's required for rendering
- Server-side rendering (SSR) requirements
- Data that should be fetched before the component mounts
- Critical path data needed for SEO

```typescript
// ✅ Good: Use loader for initial page data
export const Route = createFileRoute('/users')({
  loader: async (): Promise<{ users: User[] }> => {
    const users = await fetchUsers()
    return { users: userListSchema.parse(users) }
  },
  component: UserList,
})
```

**Prefer React Query for:**

- Data that updates frequently
- Optional/secondary data not critical for initial render
- Client-side mutations and optimistic updates
- Data that needs fine-grained cache control

```typescript
// ✅ Good: Use React Query for dynamic/optional data
function UserStats({ userId }: { userId: string }) {
  const { data: stats, isLoading } = useQuery({
    queryKey: ['user-stats', userId],
    queryFn: () => fetchUserStats(userId),
    refetchInterval: 30000, // Auto-refresh every 30s
  });

  if (isLoading) return <div>Loading stats...</div>;
  return <div>{stats?.totalPosts} posts</div>;
}
```

## Validation with Zod

**Always validate external data** - from APIs, user input, environment variables, and any data crossing system boundaries.

### Schema Organization and Best Practices

Define schemas in `src/lib/schemas.ts` with proper documentation. As the project expands, consider organizing schemas into a `src/lib/schemas/` folder structure by domain instead:

```typescript
// ✅ Good: Comprehensive schema definition
import { z } from 'zod'

export const userSchema = z.object({
  id: z.string().describe('Unique user identifier'),
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email format').optional(),
  age: z.number().int().min(13, 'Must be at least 13 years old').optional(),
  createdAt: z.string().datetime('Invalid date format'),
  role: z.enum(['admin', 'user', 'moderator']).default('user'),
})

export type User = z.infer<typeof userSchema>

// ✅ Good: Input/output schemas for transformations
export const userCreateInputSchema = userSchema.omit({
  id: true,
  createdAt: true,
})
export type UserCreateInput = z.infer<typeof userCreateInputSchema>
```

### Validation Patterns

```typescript
// ✅ Good: Safe parsing with proper error handling
function validateUserData(data: unknown): User | null {
  const result = userSchema.safeParse(data)

  if (!result.success) {
    console.error('User validation failed:', result.error.format())
    return null
  }

  return result.data
}

// ✅ Good: Server-side validation
export const ServerRoute = createServerFileRoute('/api/users').methods({
  async POST({ request }) {
    const body = await request.json()
    const result = userCreateInputSchema.safeParse(body)

    if (!result.success) {
      return new Response(
        JSON.stringify({
          error: 'Validation failed',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pomerium/mcp-app-demo](https://github.com/pomerium/mcp-app-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
