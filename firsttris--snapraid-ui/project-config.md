---
trigger: always_on
description: Full-stack SnapRAID management interface with Deno backend and React (TanStack) frontend.
---

# SnapRAID UI - Coding Standards & AI Instructions

## Project Overview
Full-stack SnapRAID management interface with Deno backend and React (TanStack) frontend.

## Tech Stack

### Backend
- **Runtime**: Deno 2.x (not Node.js!)
- **Framework**: Hono (lightweight web framework)
- **WebSockets**: Native Deno WebSocket API
- **File System**: Deno native APIs (`Deno.readTextFile`, `Deno.writeTextFile`, etc.)
- **Path handling**: `@std/path` from Deno standard library
- **Process management**: `Deno.Command` for spawning SnapRAID processes

### Frontend
- **Framework**: React 18+
- **Router**: TanStack Router (file-based routing)
- **Query**: TanStack Query for data fetching and caching
- **Build tool**: Vite
- **Styling**: Tailwind CSS (utility-first)
- **Linting**: Biome (not ESLint/Prettier)
- **Type checking**: TypeScript strict mode

### Shared
- **Language**: TypeScript throughout
- **Types**: Shared types in `/shared/types.ts`
- **API**: REST endpoints + WebSocket for real-time updates

### External Dependencies
- SnapRAID CLI (system dependency)
- Modern browser with WebSocket support

## Core Principles

### Functional Programming First
- **NO `let` statements** - Always use `const`
- **Prefer immutability** - Use spread operators, array methods
- **Arrow functions only** - No `function` keyword except for React components when needed
- **Pure functions** - Avoid side effects where possible
- **Declarative over imperative** - Use `map`, `filter`, `reduce` instead of loops

### Array Operations
```typescript
// ✅ Good - Functional
const filtered = items.filter(item => item.active);
const mapped = items.map(item => ({ ...item, processed: true }));
const found = items.find(item => item.id === targetId);

// ❌ Bad - Imperative
let result = [];
for (let i = 0; i < items.length; i++) {
  if (items[i].active) result.push(items[i]);
}
```

### Immutable Updates
```typescript
// ✅ Good - Immutable
const updated = [...items.slice(0, index), newItem, ...items.slice(index + 1)];
const modified = items.filter(item => item.id !== removeId);

// ❌ Bad - Mutation
items.splice(index, 1, newItem);
items = items.filter(...);
```

## TypeScript Standards

### Type Safety
- Always use explicit types for function parameters
- Use `type` for union types, objects
- Use `interface` only for extendable contracts
- Never use `any` - use `unknown` if type is truly unknown
- Enable strict mode

### Async/Await
```typescript
// ✅ Good
const result = await fetchData();
const items = await Promise.all(promises);

// ❌ Bad
fetchData().then(result => ...);
```

## Backend (Deno)

### File Organization
- Keep route handlers pure and simple
- Extract business logic into utility functions
- Use functional composition for complex operations
- Class methods should be arrow functions when possible

### Error Handling
```typescript
// ✅ Good
try {
  const data = await operation();
  return c.json(data);
} catch (error) {
  return c.json({ error: String(error) }, 500);
}
```

### State Management
```typescript
// ✅ Good - Encapsulated mutable state
const state = {
  value: initialValue,
};

export const updateState = (newValue: Type): void => {
  state.value = newValue;
};

// ❌ Bad - Global let
let globalValue = initialValue;
```

## Frontend (React + TanStack)

### Component Structure
- Functional components only
- Custom hooks for reusable logic
- Keep components focused and small
- Use composition over inheritance

### Props Colocation & Avoiding Props Drilling
**ALWAYS prefer moving state and queries into child components instead of passing them as props.**

```typescript
// ❌ Bad - Props drilling
function ParentPage() {
  const [filter, setFilter] = useState('all');
  const [search, setSearch] = useState('');
  const { data, isLoading, refetch } = useData();
  const mutation = useMutation();
  
  return (
    <ChildComponent 
      data={data}
      isLoading={isLoading}
      filter={filter}
      onFilterChange={setFilter}
      search={search}
      onSearchChange={setSearch}
      onRefresh={refetch}
      onAction={mutation.mutate}
    />
  );
}

// ✅ Good - State and queries colocated in child
function ParentPage() {
  const [selectedId, setSelectedId] = useState<string | null>(null);
  
  return <ChildComponent selectedId={selectedId} onSelect={setSelectedId} />;
}

function ChildComponent({ selectedId, onSelect }) {
  // State and queries live here where they're actually used
  const [filter, setFilter] = useState('all');
  const [search, setSearch] = useState('');
  const { data, isLoading, refetch } = useData();
  const mutation = useMutation();
  
  const handleAction = () => {
    mutation.mutate(selectedId);
  };
  
  // ... use state locally
}
```

**Rules for props:**
- Only pass props that are needed for coordination between components
- If data is only used in one component tree, fetch it there
- If state is only needed in one component, keep it there
- Parent components should be thin orchestrators, not data managers
- Aim for 1-3 props per component, not 10+

### State Updates
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firsttris/snapraid-ui](https://github.com/firsttris/snapraid-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
