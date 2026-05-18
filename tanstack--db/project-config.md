---
trigger: always_on
description: handleSnapshotPhase()
---

# Agent Coding Guidelines for TanStack DB

This guide provides principles and patterns for AI agents contributing to the TanStack DB codebase. These guidelines are derived from PR review patterns and reflect the quality standards expected in this project.

## Table of Contents

1. [Type Safety](#type-safety)
2. [Code Organization](#code-organization)
3. [Algorithm Efficiency](#algorithm-efficiency)
4. [Semantic Correctness](#semantic-correctness)
5. [Abstraction Design](#abstraction-design)
6. [Code Clarity](#code-clarity)
7. [Testing Requirements](#testing-requirements)
8. [Function Design](#function-design)
9. [Modern JavaScript Patterns](#modern-javascript-patterns)
10. [Edge Cases and Corner Cases](#edge-cases-and-corner-cases)

## Type Safety

### Avoid `any` Types

**❌ Bad:**

```typescript
function processData(data: any) {
  return data.value
}

const result: any = someOperation()
```

**✅ Good:**

```typescript
function processData(data: unknown) {
  if (isDataObject(data)) {
    return data.value
  }
  throw new Error('Invalid data')
}

const result: TQueryData = someOperation()
```

**Key Principles:**

- Use `unknown` instead of `any` when the type is truly unknown
- Provide proper type annotations for return values
- Use type guards to narrow `unknown` types safely
- If you find yourself using `any`, question whether there's a better type

## Code Organization

### Extract Common Logic

**❌ Bad:**

```typescript
// Duplicated logic in multiple places
function processA() {
  const key = typeof value === 'number' ? `__number__${value}` : String(value)
  // ...
}

function processB() {
  const key = typeof value === 'number' ? `__number__${value}` : String(value)
  // ...
}
```

**✅ Good:**

```typescript
function serializeKey(value: string | number): string {
  return typeof value === 'number' ? `__number__${value}` : String(value)
}

function processA() {
  const key = serializeKey(value)
  // ...
}

function processB() {
  const key = serializeKey(value)
  // ...
}
```

### Organize Utilities

**Key Principles:**

- Extract serialization/deserialization logic into utility files
- When you see identical or near-identical code blocks, extract to a helper function
- Prefer small, focused utility functions over large inline implementations
- Move reusable logic into utility modules (e.g., `utils/`, `helpers/`)

### Function Size and Complexity

**❌ Bad:**

```typescript
function syncData() {
  // 200+ lines of logic handling multiple concerns
  // - snapshot phase
  // - buffering
  // - sync state management
  // - error handling
  // all inline...
}
```

**✅ Good:**

```typescript
function syncData() {
  handleSnapshotPhase()
  manageBuffering()
  updateSyncState()
  handleErrors()
}

function handleSnapshotPhase() {
  // Focused logic for snapshot phase
}
```

**Key Principle:** If a function is massive, extract logical sections into separate functions. This improves readability and maintainability.

## Algorithm Efficiency

### Be Mindful of Time Complexity

**❌ Bad: O(n²) Queue Processing:**

```typescript
// Processes elements in queue, but elements may need multiple passes
while (queue.length > 0) {
  const job = queue.shift()
  if (hasUnmetDependencies(job)) {
    queue.push(job) // Re-queue, causing O(n²) behavior
  } else {
    processJob(job)
  }
}
```

**✅ Good: Dependency-Aware Processing:**

```typescript
// Use a data structure that respects dependencies
// Process only jobs with no unmet dependencies
// Consider topological sort for DAG-like structures
const readyJobs = jobs.filter((job) => !hasUnmetDependencies(job))
readyJobs.forEach(processJob)
```

### Use Appropriate Data Structures

**❌ Bad:**

```typescript
// O(n) lookup for each check
const items = ['foo', 'bar', 'baz' /* hundreds more */]
if (items.includes(searchValue)) {
  // ...
}
```

**✅ Good:**

```typescript
// O(1) lookup
const items = new Set(['foo', 'bar', 'baz' /* hundreds more */])
if (items.has(searchValue)) {
  // ...
}
```

**Key Principles:**

- For membership checks on large collections, use `Set` instead of `Array.includes()`
- Be aware of nested loops and their complexity implications
- Consider the worst-case scenario, especially for operations that could process many items
- Use appropriate data structures (Set for lookups, Map for key-value, etc.)

## Semantic Correctness

### Ensure Logic Matches Intent

**❌ Bad:**

```typescript
// Intending to check if subset limit is more restrictive than superset
function isLimitSubset(
  subset: number | undefined,
  superset: number | undefined,
) {
  return subset === undefined || superset === undefined || subset <= superset
}

// Problem: If subset has no limit but superset does, returns true (incorrect)
```

**✅ Good:**

```typescript
function isLimitSubset(
  subset: number | undefined,
  superset: number | undefined,
) {
  // Subset with no limit cannot be a subset of one with a limit
  return superset === undefined || (subset !== undefined && subset <= superset)
}
```

### Validate Intersections and Unions

When merging predicates or combining queries, ensure the semantics are correct:

**Example Problem:**

```sql
-- Query 1: WHERE age >= 18 LIMIT 1
-- Query 2: WHERE age >= 20 LIMIT 3
-- Naive intersection: WHERE age >= 20 LIMIT 1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TanStack/db](https://github.com/TanStack/db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
