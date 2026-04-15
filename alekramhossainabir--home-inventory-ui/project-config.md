---
trigger: always_on
description: This is a Next.js 15+ application using React 19, TypeScript, and modern web development best practices. All code generation must follow these guidelines strictly.
---

# GitHub Copilot Development Guidelines

## Project Overview
This is a Next.js 15+ application using React 19, TypeScript, and modern web development best practices. All code generation must follow these guidelines strictly.

## 1. React 19 Latest Features

### Required Usage
- **Server Components by default**: Use `"use client"` directive only when necessary (interactivity, hooks, browser APIs)
- **Server Actions**: Use Server Actions for form submissions and mutations
- **useActionState**: Replace useFormState with the new useActionState hook
- **use() hook**: For reading promises and context in components
- **useOptimistic**: For optimistic UI updates
- **useFormStatus**: For form submission states
- **ref as prop**: Pass refs directly as props instead of forwardRef
- **Enhanced Context**: Use new context improvements

### Example Pattern
```tsx
// Server Component (default)
export default async function Page() {
  const data = await fetchData();
  return <ClientComponent data={data} />;
}

// Client Component (when needed)
"use client";
export function ClientComponent({ data }) {
  const [state, dispatch] = useActionState(serverAction, initialState);
  return <form action={dispatch}>...</form>;
}
```

## 2. UX States - MANDATORY

Every data-fetching component MUST handle all three states:

### Required State Handling
```tsx
// ✅ CORRECT - All states handled
function Component() {
  const { data, isLoading, error } = useQuery(...);
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data || data.length === 0) return <EmptyState />;
  
  return <DataDisplay data={data} />;
}

// ❌ WRONG - Missing states
function Component() {
  const { data } = useQuery(...);
  return <DataDisplay data={data} />; // No loading/error/empty handling
}
```

### State Components
- **Loading**: Skeletons, spinners, or loading indicators
- **Empty**: Meaningful empty states with actions (e.g., "No items yet. Add your first item")
- **Error**: User-friendly error messages with retry options

## 3. Code Quality & Structure Standards

### File Organization
```
app/
├── (routes)/              # Route groups
│   ├── dashboard/
│   │   ├── page.tsx       # Server Component
│   │   └── loading.tsx    # Loading UI
│   └── items/
│       ├── page.tsx
│       ├── [id]/
│       │   └── page.tsx
│       └── error.tsx      # Error boundary
├── components/            # Reusable components
│   ├── ui/               # Base UI components
│   ├── forms/            # Form components
│   └── layout/           # Layout components
├── hooks/                # Custom React hooks
├── lib/                  # Utilities and configuration
│   ├── api/             # API functions
│   ├── utils/           # Helper functions
│   └── validations/     # Zod schemas
└── types/               # TypeScript type definitions
```

### Naming Conventions
- **Components**: PascalCase (e.g., `UserProfile.tsx`)
- **Hooks**: camelCase with `use` prefix (e.g., `useUserData.ts`)
- **Utilities**: camelCase (e.g., `formatDate.ts`)
- **Types**: PascalCase (e.g., `User`, `ApiResponse`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)

### Code Style
- Max function length: 50 lines
- Max file length: 300 lines
- Use early returns to reduce nesting
- Prefer composition over inheritance
- Write self-documenting code with clear variable names
- Add JSDoc comments for complex functions

## 4. Responsiveness & Accessibility

### Responsive Design (Tailwind CSS)
```tsx
// ✅ Mobile-first responsive design
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <Card className="p-4 sm:p-6" />
</div>

// Use responsive utilities: sm: md: lg: xl: 2xl:
```

### Accessibility Requirements
```tsx
// ✅ Proper semantic HTML and ARIA
<button 
  aria-label="Close dialog"
  aria-pressed={isOpen}
  disabled={isLoading}
>
  Close
</button>

<img src={url} alt="Descriptive text" />

<form aria-labelledby="form-title">
  <label htmlFor="email">Email</label>
  <input id="email" type="email" required aria-required="true" />
</form>
```

### Checklist
- [ ] Semantic HTML elements (`<nav>`, `<main>`, `<article>`, etc.)
- [ ] Proper heading hierarchy (h1 → h2 → h3)
- [ ] Alt text for images
- [ ] ARIA labels for icon buttons
- [ ] Keyboard navigation support (focus states)
- [ ] Color contrast ratio ≥ 4.5:1
- [ ] Form labels and validation messages

## 5. TypeScript - STRICT MODE

### Absolutely NO `any` Type
```tsx
// ❌ FORBIDDEN
const data: any = await fetch(...);
function process(item: any) { }

// ✅ CORRECT - Use proper types
interface Item {
  id: string;
  name: string;
  quantity: number;
}

const data: Item[] = await fetch(...);
function process(item: Item): void { }

// ✅ For unknown data, use unknown and narrow
const data: unknown = await fetch(...);
if (isItem(data)) {
  process(data); // TypeScript knows data is Item
}
```

### Type Best Practices
- Define interfaces for all data structures
- Use `type` for unions/intersections, `interface` for objects
- Leverage utility types: `Partial<T>`, `Pick<T>`, `Omit<T>`, `Required<T>`
- Use `as const` for literal types
- Define return types explicitly for functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlEkramHossainAbir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
