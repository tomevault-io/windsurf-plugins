---
trigger: always_on
description: Apply the [general coding guidelines](./general-coding.instructions.md) to all code.
---


# Project coding standards for TypeScript and React

Apply the [general coding guidelines](./general-coding.instructions.md) to all code.

## TypeScript Guidelines

- Use TypeScript for all new code
- Follow functional programming principles where possible
- Use interfaces for data structures and type definitions
- Use optional chaining (?.) and nullish coalescing (??) operators
- No using `any` type; use specific types or generics

## React Guidelines

- Use functional components with hooks
- Follow the React hooks rules (no conditional hooks)
- Use React.FC type for components with children
- **MANDATORY: Keep components small and focused**
  - **Maximum 300 lines per component file**
  - **Single responsibility principle - one main purpose per component**
  - **Extract sub-components when logic becomes complex**
  - **Break large components into smaller, composable pieces**
  - **Use custom hooks to extract complex logic**

## ANTI-PATTERNS TO AVOID

### ❌ FORBIDDEN: Useless Wrapper Hooks

**Never create hooks that just wrap other hooks without adding value.**

```typescript
// ❌ BAD - Useless wrapper that adds no value
export const useCategoryData = () => {
  const categoryTreeQuery = useCategoryTree();
  const categoriesQuery = useCategories();
  const createMutation = useCreateCategoryMutation();

  return {
    categories: categoriesQuery.data || [],
    categoryTree: categoryTreeQuery.data || [],
    isLoading: categoryTreeQuery.isLoading || categoriesQuery.isLoading,
    createCategory: createMutation.mutateAsync,
    // Just wrapping existing hooks with different names
  };
};

// ✅ GOOD - Use trpc/React Query hooks directly
export function CategoryManagement() {
  const { data: categories, isLoading } = useCategories();
  const { data: categoryTree } = useCategoryTree();
  const createMutation = useCreateCategoryMutation();

  // Direct usage - clear, maintainable, no extra layers
}
```

### ❌ FORBIDDEN: Unnecessary Abstraction Layers

```typescript
// ❌ BAD - Wrapping functions that don't need wrapping
export const categoryService = {
  getAll: () => api.category.getAll.query(),
  create: (data) => api.category.create.mutate(data),
  // Just wrapping trpc calls
};

// ✅ GOOD - Use trpc client directly
export function MyComponent() {
  const { data } = api.category.getAll.useQuery();
  const createMutation = api.category.create.useMutation();
}
```

### When Wrapper Hooks ARE Justified:

**Only create wrapper hooks when they add real value:**

```typescript
// ✅ GOOD - Adds business logic and validation
export const useCategoryWithValidation = (categoryId: string) => {
  const { data: category, ...query } = useCategory(categoryId);

  // ADDS VALUE: Complex validation logic
  const validationErrors = useMemo(() => {
    if (!category) return [];
    const errors = [];
    if (!category.name?.trim()) errors.push("Name is required");
    if (category.name.length > 50) errors.push("Name too long");
    return errors;
  }, [category]);

  // ADDS VALUE: Computed state
  const isValid = validationErrors.length === 0;

  return {
    category,
    validationErrors,
    isValid,
    ...query,
  };
};

// ✅ GOOD - Combines multiple queries with complex logic
export const useDashboardData = () => {
  const { data: stats } = useCategoryStats();
  const { data: recent } = useRecentActivities();

  // ADDS VALUE: Complex data transformation
  const dashboardMetrics = useMemo(() => {
    if (!stats || !recent) return null;

    return {
      productivity: calculateProductivity(stats, recent),
      trends: analyzeTrends(recent),
      alerts: generateAlerts(stats),
    };
  }, [stats, recent]);

  return { dashboardMetrics, isLoading: !stats || !recent };
};
```

## Component Size and Structure Enforcement

### MANDATORY RULES:

1. **File Size Limit**: No component file should exceed 300 lines
2. **Single Responsibility**: Each component should have one clear purpose
3. **Extract When Complex**: If a component has more than 3-4 different concerns, split it
4. **Use Composition**: Prefer multiple small components over one large component

### Component Breakdown Patterns:

```typescript
// ❌ BAD - Large monolithic component (400+ lines)
export function LargeDataTable() {
  // 50+ lines of state management
  // 100+ lines of data processing
  // 200+ lines of JSX rendering
  // Multiple different concerns mixed together
}

// ✅ GOOD - Broken into focused components
export function DataTable() {
  const data = useTableData();
  const { filters, setFilters } = useTableFilters();
  const { pagination, setPagination } = useTablePagination();

  return (
    <div>
      <TableFilters filters={filters} onFiltersChange={setFilters} />
      <TableContent data={data} pagination={pagination} />
      <TablePagination pagination={pagination} onPaginationChange={setPagination} />
    </div>
  );
}

// Individual focused components in separate files:
// - TableFilters.tsx (handles only filtering logic)
// - TableContent.tsx (handles only data display)
// - TablePagination.tsx (handles only pagination)
// - useTableData.ts (custom hook for data logic)
// - useTableFilters.ts (custom hook for filter logic)
// - useTablePagination.ts (custom hook for pagination logic)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itracksy/itracksy](https://github.com/itracksy/itracksy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
