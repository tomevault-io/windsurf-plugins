---
trigger: always_on
description: The way to link PRs to an issue is via changing the root comment in a PR to reference the issue number. i.e.:
---

# PR Linking
The way to link PRs to an issue is via changing the root comment in a PR to reference the issue number. i.e.:

| Method | API | Auto-closes? | UI-equivalent sidebar link? |
|--------|-----|--------------|----------------------------|
| Fixes #123 in PR body | REST | ✅ Yes | ✅ Appears under "Linked issues" |
| "Link issue" via sidebar | GraphQL (internal) | ❌ No | ✅ Appears under "Linked issues" |
| Add comment with #123 | REST | ❌ No | ✅ Appears in timeline only |
| Add assignee via /assignees | REST | ❌ No | ❌ Only changes assignees |

# Optimistic updates

This codebase uses an **optimistic update pattern** for all GitHub API operations. This pattern ensures instant UI feedback despite GitHub's API indexing delays (2-5+ seconds for GraphQL changes).

## The Core Problem

GitHub's APIs have significant delays:
- **REST API**: Immediate for reads, but writes may not be reflected in subsequent reads
- **GraphQL API**: Has indexing delays for derived data (e.g., `closedByPullRequestsReferences`)
- **Result**: If you update GitHub and immediately refetch, you get stale data

❌ **Anti-pattern**: Update GitHub → Wait for API → Update UI (causes flickering)
✅ **Solution**: Update GitHub → Update UI from local cache → Eventually sync with GitHub

## The Four-Step Pattern

### Step 1: Set Loading State on the Object

```typescript
const key = buildObjectKey(params); // e.g., `${owner}/${repo}#${number}`
const object = get().objects.get(key);

// Set loading flag on the object itself, not a separate variable
set((state) => {
  const newObjects = new Map(state.objects);
  newObjects.set(key, { ...object, isPerformingOperation: true });
  return { objects: newObjects };
});
```

**Key principles:**
- Loading state lives ON the object (`object.isPerformingOperation`)
- NOT in separate variables (`const [isLoading, setIsLoading]`)
- Prevents race conditions when operating on multiple objects

### Step 2: Make the API Call

```typescript
try {
  const api = new GitHubAPI(token);
  await api.performOperation(params);
  console.log('✅ API operation successful');
```

**Key principles:**
- Use try/catch for error handling
- Log success for debugging
- Don't await subsequent updates (let them happen in background)

### Step 3: Update Store from Local Cache (Not GitHub)

```typescript
  // Build new state from YOUR stores, not from GitHub API response
  const relatedStore = useRelatedStore.getState();
  const newData = buildFromLocalCache(relatedStore, params);

  // Update object with new data + clear loading flag
  set((state) => {
    const newObjects = new Map(state.objects);
    const currentObject = newObjects.get(key);
    if (currentObject) {
      newObjects.set(key, {
        ...currentObject,
        data: newData,                    // Updated data from local cache
        isPerformingOperation: false,     // Clear loading flag
      });
    }
    return { objects: newObjects };
  });
```

**Key principles:**
- Build new state from **your local stores**, not from GitHub API
- Your stores are the source of truth (they're already up-to-date)
- Clear the loading flag in the same update

### Step 4: Error Handling - Revert Loading Only

```typescript
} catch (error) {
  console.error('❌ Operation failed:', error);
  
  // Only revert the loading flag, don't change data
  set((state) => {
    const newObjects = new Map(state.objects);
    const currentObject = newObjects.get(key);
    if (currentObject) {
      newObjects.set(key, { 
        ...currentObject, 
        isPerformingOperation: false 
      });
    }
    return { objects: newObjects };
  });
  
  throw error; // Re-throw for caller to handle
}
```

**Key principles:**
- Only revert the loading flag
- Don't change the data (user sees what they tried to do)
- Re-throw error for upstream handling

## Critical Implementation Details

### 1. Loading State on Objects, Not Separately

```typescript
// ❌ BAD: Separate loading state
const [isUpdating, setIsUpdating] = useState(false);
// Problem: Doesn't scale, race conditions with multiple operations

// ✅ GOOD: Loading state on object
interface MyObject {
  id: number;
  data: string;
  isPerformingOperation?: boolean;  // Per-object loading flag
}
```

### 2. Check for `undefined`, Not Truthiness

```typescript
// ❌ BAD: Empty arrays are falsy
const data = object.data?.length ? object.data : fallback;
// Problem: [] is falsy, triggers fallback incorrectly

// ✅ GOOD: Check if property exists
const data = object.data !== undefined ? object.data : fallback;
// Works correctly: undefined → fallback, [] → use it
```

### 3. Preserve State in Updates

When updating objects, preserve loading flags and optional data:

```typescript
const updatedObject = {
  ...newObject,
  // Preserve optional fields if not explicitly provided
  optionalData: newObject.optionalData ?? existingObject?.optionalData ?? [],
  isPerformingOperation: newObject.isPerformingOperation ?? existingObject?.isPerformingOperation ?? false,
};
```

### 4. Close Modals Immediately

```typescript
// In the component that opens the modal:
const handleUpdate = async (params) => {
  // Close modal FIRST (optimistic)
  handleCloseModal();
  
  // Then perform operation in background

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [areibman/bottleneck](https://github.com/areibman/bottleneck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
