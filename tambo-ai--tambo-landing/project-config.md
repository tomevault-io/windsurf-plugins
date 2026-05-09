---
trigger: always_on
description: Provides an `AbortSignal` that triggers when the component's cache scope expires.
---

---
description: Project overview and cross-cutting concerns
globs: *.tsx, *.jsx, *.css, *.js, *.ts
---

# Satus Project Guidelines

## Technology Stack

- **Next.js 16.0.1** - App Router with Turbopack support and Cache Components
- **React 19.2.0** - Latest features including `<Activity />`, `useEffectEvent`, and `cacheSignal`
- **React Compiler enabled** - Automatically optimizes most component re-renders and memoization; manual memoization is rarely needed
- **TypeScript** - Strict mode enabled
- **Tailwind CSS 4.1.16** - CSS-first configuration
- **Biome 2.3.3** - Linting and formatting
- **Bun** - JavaScript runtime and package manager

## React 19.2 New Features

### 1. `<Activity />` Component
Manage off-screen component visibility and defer updates for better performance.

```tsx
import { Activity } from 'react'

// Hide tab content when not visible
<Activity mode={isActive ? 'visible' : 'hidden'}>
  <ExpensiveComponent />
</Activity>
```

**Use Cases:**
- Tab systems or carousels
- Off-screen WebGL scenes (3D graphics, shaders)
- Accordion components
- Drawer/modal systems
- Image galleries and carousels

**Benefits:**
- Pre-render content without performance impact
- Automatic effect cleanup when hidden
- Better resource management for complex UIs

### 2. `useEffectEvent` Hook
Separate event logic from effect dependencies to prevent unnecessary re-runs.

```tsx
import { useEffect, useEffectEvent } from 'react'

function Component({ url, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme) // theme changes won't trigger reconnect
  })

  useEffect(() => {
    const connection = createConnection(url)
    connection.on('connected', onConnected)
    connection.connect()
    return () => connection.disconnect()
  }, [url]) // Only reconnect when url changes
}
```

**Use Cases:**
- Complex event handlers with multiple dependencies
- Scroll/transform callbacks
- WebGL mouse/interaction handlers
- Animation callbacks

**Benefits:**
- Reduces unnecessary effect re-runs
- Cleaner dependency arrays
- Better separation of concerns

### 3. `cacheSignal` (Server Components Only)
Provides an `AbortSignal` that triggers when the component's cache scope expires.

```tsx
import { cacheSignal } from 'react'

async function fetchUserData(id: string) {
  const signal = cacheSignal() // Auto-aborts on cache expiry
  const response = await fetch(`/api/users/${id}`, { signal })
  return response.json()
}
```

**Use Cases:**
- Sanity CMS queries
- Shopify API calls
- Any server component data fetching
- Replace custom timeout logic with automatic cleanup

**Benefits:**
- Automatic cleanup of stale requests
- Better resource management
- Simpler than manual AbortController

### 4. Performance Tracks in Chrome DevTools
React 19.2 integrates custom performance tracks into Chrome DevTools:
- **Scheduler Track:** Displays React's workload prioritization
- **Components Track:** Shows component hierarchy and timing

**Usage:** Open Chrome DevTools → Performance tab → Record a profile → Look for React-specific tracks

## File Organization

```
├── app/                  # Next.js pages and routes
├── components/           # Reusable UI components
├── hooks/                # Custom React hooks
├── integrations/         # Third-party integrations
├── libs/                 # Utility libraries
│   ├── cleanup-integrations.ts  # Remove unused integrations
│   ├── fetch-with-timeout.ts    # API resilience
│   ├── metadata.ts              # SEO/metadata helpers
│   ├── validate-env.ts          # Environment validation
│   └── ...
├── orchestra/            # Debug tools (dev-only)
├── styles/               # Styling configuration
└── webgl/                # WebGL and 3D graphics
```

## Cross-Cutting Concerns

### React Compiler & Memoization

**React Compiler is enabled and handles ALL optimization automatically.**

- **DO NOT use `useMemo`, `useCallback`, or `React.memo` in new code.**
- The compiler optimizes all component re-renders, memoization, and dependencies automatically.
- Only use manual memoization if you encounter a proven edge case where the compiler cannot optimize (extremely rare).
- If you see these in existing code, they can likely be removed safely.
- **CRITICAL EXCEPTION: Use `useRef` for object instantiation** - Creating new object instances on every render creates new references that trigger effects, causing infinite loops.
- Refer to the [React Compiler documentation](https://react.dev/reference/react/compiler) for edge cases.

```tsx
// ❌ DON'T: Manual memoization for simple calculations (compiler handles this)
const memoizedValue = useMemo(() => computeExpensive(a, b), [a, b])
const memoizedCallback = useCallback(() => doSomething(a), [a])

// ✅ DO: Let React Compiler optimize automatically
const value = computeExpensive(a, b)
const handleClick = () => doSomething(a)

// ⚠️ EXCEPTION: Object instantiation MUST use useRef
// ❌ DON'T: This causes infinite re-renders when passed to effects/deps
const instance = new SomeClass()

// ✅ DO: Use useRef for object instantiation
const instanceRef = useRef<SomeClass | null>(null)
if (!instanceRef.current) {
  instanceRef.current = new SomeClass(params)
}
const instance = instanceRef.current
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tambo-ai/tambo-landing](https://github.com/tambo-ai/tambo-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
