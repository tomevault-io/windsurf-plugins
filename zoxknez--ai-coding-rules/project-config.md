---
trigger: always_on
description: USE WHEN: working with Next.js 15 App Router, React Server Components, or Server Actions.
---


# Next.js 15 App Router Rules

> This module covers Next.js 15+ with App Router. For Pages Router, use general React rules.

## Component Strategy

### Server vs Client Components

| Type | Default | Directive | Use When |
|------|---------|-----------|----------|
| Server Component | ✅ Yes | None needed | Data fetching, no interactivity |
| Client Component | No | `'use client'` | Hooks, events, browser APIs |

### Decision Tree

```
Does this component need...
├── useState/useEffect/useContext? → Client Component
├── onClick/onChange/onSubmit? → Client Component  
├── window/document/localStorage? → Client Component
├── Only data fetching? → Server Component ✅
└── Only rendering props? → Server Component ✅
```

### Placement Rules

```typescript
// ✅ Server Component (default)
// No directive needed
async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId); // Direct DB/API call
  return <div>{user.name}</div>;
}

// ✅ Client Component (when needed)
'use client';

import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

## Data Fetching

### Parallel Fetching (CRITICAL)

```typescript
// ❌ BAD: Waterfall — each awaits the previous
async function Page() {
  const user = await getUser();       // 100ms
  const posts = await getPosts();     // 100ms
  const comments = await getComments(); // 100ms
  // Total: 300ms
}

// ✅ GOOD: Parallel — all run simultaneously
async function Page() {
  const [user, posts, comments] = await Promise.all([
    getUser(),      // 100ms
    getPosts(),     // 100ms  
    getComments()   // 100ms
  ]);
  // Total: ~100ms
}
```

### Streaming with Suspense

```typescript
import { Suspense } from 'react';

async function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* Instantly visible */}
      
      <Suspense fallback={<LoadingSkeleton />}>
        <SlowComponent />
        {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

---

## Server Actions

### Form Mutations

```typescript
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  // Validation
  if (!title || title.length < 3) {
    return { error: 'Title must be at least 3 characters' };
  }
  
  // Database operation
  await db.insert(posts).values({ title, content });
  
  // Revalidate and redirect
  revalidatePath('/posts');
  redirect('/posts');
}
```

### Using in Components

```typescript
// ✅ Form with Server Action
import { createPost } from './actions';

function CreatePostForm() {
  return (
    <form action={createPost}>
      <input name="title" required minLength={3} />
      <textarea name="content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### With useFormState (Client Feedback)

```typescript
'use client';

import { useFormState } from 'react-dom';
import { createPost } from './actions';

function CreatePostForm() {
  const [state, formAction] = useFormState(createPost, null);
  
  return (
    <form action={formAction}>
      <input name="title" />
      {state?.error && <p className="text-red-500">{state.error}</p>}
      <button type="submit">Create</button>
    </form>
  );
}
```

---

## Caching & Revalidation

### Cache Options

```typescript
// Default: Cached indefinitely (like getStaticProps)
fetch(url);

// Time-based revalidation (ISR)
fetch(url, { next: { revalidate: 60 } }); // Refresh every 60s

// No cache (like getServerSideProps)
fetch(url, { cache: 'no-store' });

// Tagged cache (for manual revalidation)
fetch(url, { next: { tags: ['posts'] } });
```

### Manual Revalidation

```typescript
import { revalidatePath, revalidateTag } from 'next/cache';

// Revalidate specific path
revalidatePath('/posts');

// Revalidate by tag
revalidateTag('posts');

// Revalidate layout and all child pages
revalidatePath('/dashboard', 'layout');
```

---

## File Conventions

### Special Files

| File | Purpose |
|------|---------|
| `page.tsx` | Route UI |
| `layout.tsx` | Shared layout (persists across navigations) |
| `loading.tsx` | Loading UI (automatic Suspense boundary) |
| `error.tsx` | Error UI (automatic Error boundary) |
| `not-found.tsx` | 404 UI |
| `route.ts` | API endpoint |

### Route Groups

```
app/
├── (marketing)/     # Group: doesn't affect URL
│   ├── about/
│   └── contact/
├── (dashboard)/     # Group: different layout
│   ├── settings/
│   └── profile/
└── layout.tsx       # Root layout
```

### Dynamic Routes

```
app/
├── posts/
│   ├── [slug]/           # Dynamic: /posts/my-post
│   │   └── page.tsx
│   ├── [...slug]/        # Catch-all: /posts/a/b/c
│   │   └── page.tsx
│   └── [[...slug]]/      # Optional catch-all: /posts or /posts/a/b
│       └── page.tsx
```

---

## Asset Optimization

### Images

```typescript
import Image from 'next/image';

// ✅ Always use next/image
<Image 
  src="/hero.jpg"
  alt="Hero image"
  width={1200}
  height={600}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
