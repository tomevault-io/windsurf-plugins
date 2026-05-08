---
trigger: always_on
description: React, TypeScript, and web development best practices for performance optimization
---


# React Best Practices

Comprehensive performance optimization guide for React applications. Contains 35+ rules across 7 categories, prioritized by impact from critical (eliminating waterfalls, reducing bundle size) to incremental (advanced patterns). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations.

---

## 1. Eliminating Waterfalls

**Impact: CRITICAL**

Waterfalls are the #1 performance killer. Each sequential await adds full network latency. Eliminating them yields the largest gains.

### 1.1 Defer Await Until Needed

**Impact: HIGH (avoids blocking unused code paths)**

Move `await` operations into the branches where they're actually used to avoid blocking code paths that don't need them.

**Incorrect: blocks both branches**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // Returns immediately but still waited for userData
    return { skipped: true }
  }
  
  // Only this branch uses userData
  return processUserData(userData)
}
```

**Correct: only blocks when needed**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // Returns immediately without waiting
    return { skipped: true }
  }
  
  // Fetch only when needed
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

**Another example: early return optimization**

```typescript
// Incorrect: always fetches permissions
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId)
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}

// Correct: fetches only when needed
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  const permissions = await fetchPermissions(userId)
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}
```

This optimization is especially valuable when the skipped branch is frequently taken, or when the deferred operation is expensive.

### 1.2 Promise.all() for Independent Operations

**Impact: CRITICAL (2-10× improvement)**

When async operations have no interdependencies, execute them concurrently using `Promise.all()`.

**Incorrect: sequential execution, 3 round trips**

```typescript
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()
```

**Correct: parallel execution, 1 round trip**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```

### 1.3 Dependency-Based Parallelization

**Impact: CRITICAL (2-10× improvement)**

For operations with partial dependencies, start independent operations immediately, even if you don't await them yet.

**Incorrect: config waits for auth, data waits for both**

```typescript
async function fetchData() {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchUserData(session.user.id)
  return { data, config }
}
```

**Correct: auth and config start immediately**

```typescript
async function fetchData() {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise
  const [config, data] = await Promise.all([
    configPromise,
    fetchUserData(session.user.id)
  ])
  return { data, config }
}
```

### 1.4 Strategic Suspense Boundaries

**Impact: HIGH (faster initial paint)**

Use Suspense boundaries to show the wrapper UI faster while data loads.

**Incorrect: wrapper blocked by data fetching**

```tsx
function Page() {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    fetchData().then(d => {
      setData(d)
      setLoading(false)
    })
  }, [])
  
  if (loading) return <Skeleton />
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

The entire layout waits for data even though only the middle section needs it.

**Correct: wrapper shows immediately, data streams in**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

function DataDisplay() {
  const data = use(fetchData()) // Using React's use() hook
  return <div>{data.content}</div>
}
```

Sidebar, Header, and Footer render immediately. Only DataDisplay waits for data.

**When NOT to use this pattern:**

- Critical data needed for layout decisions (affects positioning)
- Small, fast queries where suspense overhead isn't worth it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataCovey/nornweave](https://github.com/DataCovey/nornweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
