---
trigger: always_on
description: Modern web development: JavaScript, TypeScript, React, Next.js, Vue, Node.js, HTML/CSS, Tailwind CSS, and web APIs
---


# Web Development Patterns

Modern web development best practices for JavaScript, TypeScript, and related frameworks.

## Version Policy

Never hardcode fast-moving framework versions in rules files.

Before using a new framework or package:

```text
1. Search the current version with the actual month and year
2. Check compatibility with the existing stack
3. Then recommend the install or setup path
```

For packages already present in `package.json`, trust the repo unless errors suggest a version mismatch.

Do not describe setup advice as current or recommended unless it is backed by current authoritative sources.

## Project Setup Workflow

For new web projects, use the framework's official CLI or official package-manager `create` path instead of manual scaffolding:

```bash
# Next.js
npx create-next-app@latest my-app --typescript --tailwind --app --eslint

# React (Vite)
npm create vite@latest my-app -- --template react-ts

# Vue
npm create vue@latest
```

For component libraries or framework add-ons, check current setup guidance before recommending commands.

Do not hand-write project manifests, boilerplate, or generated folder structure when the framework provides an official scaffold.

## Post-Scaffold Acceptance

After creating a new web app or making structural setup changes, prove the scaffold actually works before claiming success:

```text
1. Dependency install succeeds
2. Dev server or framework health check starts successfully
3. Production build succeeds
4. One primary happy-path flow works end to end
5. Promised integrations actually work: styling, routing, persistence, auth, or other claimed layers
6. No obvious console or runtime errors appear on the happy path
```

If Tailwind, shadcn, PostCSS, or another styling pipeline is part of the promise, verify that styles are actually processed instead of assuming the scaffold wired them correctly.

## Verification Guidance

After meaningful web changes, run the smallest useful check for the task:
- lint or typecheck for localized edits
- build for integration-sensitive changes
- focused tests for behavior changes
- browser verification for layout and interaction work
- for new apps or scaffolds, do not stop at static checks; verify startup, build, and one user-visible flow

---

## JavaScript Best Practices

### Modern Syntax
- Use `const` by default, `let` when reassignment needed, never `var`
- Prefer arrow functions for callbacks and short functions
- Use template literals for string interpolation
- Destructure objects and arrays
- Use spread operator for immutable operations

```javascript
// Modern patterns
const { name, email } = user;
const updatedList = [...items, newItem];
const greeting = `Hello, ${name}!`;
const fetchData = async () => { /* ... */ };
```

### Async Patterns
- Prefer `async/await` over `.then()` chains
- Always handle errors with try/catch
- Use `Promise.all()` for parallel operations
- Use `Promise.allSettled()` when some failures are acceptable

```javascript
// Parallel operations
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts()
]);

// Error handling
try {
  const data = await fetchData();
} catch (error) {
  console.error('Fetch failed:', error.message);
  throw new DataFetchError(error);
}
```

### Array Methods
- `map()` for transformations
- `filter()` for selection
- `reduce()` for aggregation
- `find()` for single item lookup
- Avoid mutating methods on shared data

---

## TypeScript Best Practices

### Type Definitions
- Use interfaces for object shapes
- Use types for unions, intersections, and computed types
- Prefer `unknown` over `any`
- Use `as const` for literal types

```typescript
// Interface for objects
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

// Type for unions
type Result<T> = { success: true; data: T } | { success: false; error: string };

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### Type Safety
- Enable strict mode in tsconfig
- Avoid type assertions unless necessary
- Use generics for reusable code
- Leverage inference when types are obvious

### Utility Types
```typescript
Partial<T>     // All properties optional
Required<T>    // All properties required
Pick<T, K>     // Select specific properties
Omit<T, K>     // Exclude specific properties
Record<K, V>   // Object with keys K and values V
```

---

## React Patterns

### Component Structure
```tsx
// Recommended component structure
interface Props {
  // Props interface at top
}

export function ComponentName({ prop1, prop2 }: Props) {
  // Hooks first
  const [state, setState] = useState();
  const ref = useRef();
  
  // Derived values
  const computed = useMemo(() => /* ... */, [deps]);
  
  // Effects
  useEffect(() => { /* ... */ }, [deps]);
  
  // Handlers
  const handleClick = () => { /* ... */ };
  
  // Render
  return (/* JSX */);
}
```

### Hooks Best Practices

#### useState
```tsx
// Group related state
const [form, setForm] = useState({ name: '', email: '' });

// Use updater function for derived state
setCount(prev => prev + 1);
```

#### useEffect
```tsx
// Cleanup subscriptions
useEffect(() => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
