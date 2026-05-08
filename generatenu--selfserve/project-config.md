---
trigger: always_on
description: routes/        # File-based routing (TanStack Router)
---

# SelfServe

## Project Structure

```
clients/
  web/src/
    routes/        # File-based routing (TanStack Router)
    components/    # Feature-organized React components (ui/, home/, requests/, rooms/, profile/)
    hooks/         # Custom React hooks
    lib/           # Utilities (cn helper, etc.)
    tests/         # Test files
  shared/src/
    api/           # HTTP client and config
    types/         # Shared API types
  mobile/          # React Native app (mirrors web structure)
backend/
  cmd/             # Entry points
  config/          # Config loading
  internal/
    handler/       # HTTP handlers (Fiber)
    repository/    # Database access (pgx)
    models/        # Domain models with validation tags
    service/       # Business logic and app initialization
```

## Frontend Patterns

### File Naming

- Components: PascalCase (`Button.tsx`, `HomeHeader.tsx`)
- Hooks: kebab-case with `use-` prefix (`use-dropdown.ts`)
- Utilities: camelCase (`utils.ts`)
- Routes: kebab-case with `.` segments (`_protected/rooms.index.tsx`)
- Layout routes: underscore prefix (`__root.tsx`, `_protected.tsx`)

### Component Structure

Functional components only. Props typed with `type` or `interface`. Single named export per file.

```tsx
type ButtonVariant = "primary" | "secondary";
type ButtonProps = ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: ButtonVariant;
};

const variantStyles: Record<ButtonVariant, string> = {
  primary: "bg-primary text-white hover:bg-primary-hover",
  secondary: "bg-bg-container text-text-default hover:bg-bg-selected",
};

export function Button({ variant = "secondary", className = "", ...props }: ButtonProps) {
  return (
    <button
      className={cn(variantStyles[variant], className)}
      {...props}
    />
  );
}
```

### Routing (TanStack Router)

File-based routing. Route components exported as `Route` const. Dynamic segments use `$` prefix.

```tsx
export const Route = createFileRoute("/_protected/home")({
  component: HomePage,
});

function HomePage() {
  // implementation
}
```

Protected routes use the `_protected` layout route. Dynamic routes: `guests.$guestId.tsx`.

### Styling (Tailwind CSS)

Utility-first with no custom CSS components. Use `cn()` for conditional/merged classes.

```tsx
import { cn } from "@/lib/utils";

// Variant map pattern
const variantStyles: Record<Variant, string> = { ... };

// Merge classes
<div className={cn("base-class", isActive && "active-class", className)} />
```

**Custom theme tokens** (use these instead of raw colors):
- Text: `text-text-default`, `text-text-subtle`, `text-text-secondary`
- Background: `bg-bg-primary`, `bg-bg-container`, `bg-bg-selected`, `bg-bg-disabled`
- Stroke: `stroke-default`, `stroke-subtle`, `stroke-disabled`
- Brand: `bg-primary`, `hover:bg-primary-hover`
- Status: success, warning, danger, info variants

### Imports

Use path aliases — never relative imports that traverse directories.

```ts
import { Button } from "@/components/ui/Button";
import { request } from "@shared/api/client";
import type { User } from "@shared/types/api.types";
```

Named exports only. Default exports are only acceptable in generated code.

### Data Fetching (TanStack Query)

QueryClient defaults: `staleTime: 60_000`, `retry: 1`, `refetchOnWindowFocus: false`.

Query key convention: `["resource", id]`.

```ts
const { data } = useQuery({
  queryKey: ["rooms", hotelId],
  queryFn: () => getRooms(hotelId),
});
```

### Mutations (TanStack Query)

```ts
const { mutate } = useMutation({
  mutationFn: (value: string) => request({ url: `/resource/${id}`, method: "PUT", data: { field: value } }),
  onSuccess: (result) => { /* handle success */ },
  onError: (error) => { /* handle error */ },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ["resource", id] });
  },
});
```

### Optimistic Updates (TanStack Query)

Use this pattern for mutations that should update the UI immediately before the server responds.

```ts
const { mutate } = useMutation({
  mutationFn: (value: string) =>
    request({ url: `/resource/${id}`, method: "PUT", data: { field: value } }),
  onMutate: async (value) => {
    // Cancel any in-flight refetches to avoid overwriting optimistic update
    await queryClient.cancelQueries({ queryKey: ["resource", id] });
    // Snapshot previous value for rollback
    const previous = queryClient.getQueryData<ResourceType>(["resource", id]);
    // Optimistically update the cache
    queryClient.setQueryData<ResourceType>(["resource", id], (old) => ({
      ...old,
      field: value,
    }));
    return { previous };
  },
  onError: (_err, _vars, context) => {
    // Roll back to snapshot on failure
    queryClient.setQueryData(["resource", id], context?.previous);
  },
  onSettled: () => {
    // Always resync with server after mutation
    queryClient.invalidateQueries({ queryKey: ["resource", id] });
  },
});
```

### Custom Hooks

Follow `use*` naming. Generic hooks accept type parameters.

```ts
export const useDropdown = <T>({ selected, onChangeSelectedItems }: UseDropdownOptions<T>) => {
  // implementation
};
```

### Authentication (Clerk)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GenerateNU/selfserve](https://github.com/GenerateNU/selfserve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
