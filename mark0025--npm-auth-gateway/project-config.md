---
trigger: always_on
description: Use when:
---

Below is a **Cursor rule aligned with the official Next.js 16.1 caching model** based directly on the documentation you provided.
It enforces the **correct modern approach**:

* `fetch` caching
* `use cache`
* `cacheTag`
* `revalidateTag('tag','max')`
* `updateTag`
* `revalidatePath`
* **avoiding `unstable_cache` unless legacy**

This prevents Cursor from generating **old Next.js 13/14 caching patterns**.

Place this rule at:

```
.cursor/rules/nextjs16-caching-and-revalidation.mdc
```

---

# `.cursor/rules/nextjs16-caching-and-revalidation.mdc`

````mdc
---
description: "Defines correct caching and revalidation patterns for Next.js 16 using fetch, Cache Components, cacheTag, revalidateTag, updateTag, and revalidatePath."
alwaysApply: false
---

# Next.js 16 Caching and Revalidation Architecture

This rule defines how caching and revalidation must be implemented in Next.js 16 applications.

Next.js provides several APIs for caching and invalidating cached data:

- fetch
- use cache
- cacheTag
- revalidateTag
- updateTag
- revalidatePath

Legacy API:
- unstable_cache (should generally be avoided)

The goal is to:

- minimize repeated work
- reduce database load
- enable partial data updates
- support stale-while-revalidate patterns

---

# 1. Default Fetch Behavior

By default, fetch requests are **not cached**.

Example:

```tsx
const data = await fetch("https://api.example.com/data")
````

To cache the request, set:

```tsx
const data = await fetch("https://api.example.com/data", {
  cache: "force-cache"
})
```

Rules:

Use `force-cache` when:

* data changes infrequently
* API responses can be reused
* rendering static pages

Do not cache requests that contain:

* authentication tokens
* user-specific data
* session data

---

# 2. Time-Based Revalidation

Fetch supports time-based revalidation using:

```
next.revalidate
```

Example:

```tsx
const data = await fetch("https://api.example.com/products", {
  next: { revalidate: 3600 }
})
```

Meaning:

```
cache result
revalidate every 3600 seconds
```

Use this for:

* CMS content
* product listings
* blog posts
* marketing pages

---

# 3. Tagging Fetch Requests

Fetch requests may include **cache tags**.

Example:

```tsx
const data = await fetch("https://api.example.com/users", {
  next: {
    tags: ["users"]
  }
})
```

Tags allow cache invalidation across multiple resources.

Example use case:

```
getUsers()
getUserById()
getUserPosts()
```

All can share the same tag:

```
users
```

When the tag is invalidated, all cached entries refresh.

---

# 4. Cache Components With "use cache"

Next.js 16 introduced **Cache Components**.

This allows caching **any server computation**, not only fetch.

Example:

```tsx
import { cacheTag } from "next/cache"

export async function getProducts() {
  "use cache"

  cacheTag("products")

  const products = await db.query("SELECT * FROM products")

  return products
}
```

Rules:

Use `"use cache"` for:

* database queries
* expensive computations
* filesystem reads
* external APIs

This replaces many previous `unstable_cache` usages.

---

# 5. Using cacheTag

`cacheTag` associates cached work with a tag.

Example:

```tsx
import { cacheTag } from "next/cache"

export async function getPosts() {
  "use cache"

  cacheTag("posts")

  return db.post.findMany()
}
```

Tags enable centralized invalidation.

Multiple functions can reuse the same tag.

---

# 6. Revalidating Cache With revalidateTag

`revalidateTag` is used after a mutation to refresh cached data.

Example:

```tsx
import { revalidateTag } from "next/cache"

export async function updateUser(id: string) {
  await db.user.update({ where: { id } })

  revalidateTag("users", "max")
}
```

Rules:

Always use:

```
revalidateTag(tag, "max")
```

Reason:

This enables **stale-while-revalidate** behavior:

* users see cached content
* background refresh occurs
* avoids blocking requests

Never omit `"max"` unless legacy behavior is required.

---

# 7. Immediate Cache Expiration With updateTag

`updateTag` is designed specifically for **Server Actions**.

Example:

```tsx
import { updateTag } from "next/cache"

export async function createPost(formData: FormData) {
  const post = await db.post.create({
    data: {
      title: formData.get("title"),
      content: formData.get("content")
    }
  })

  updateTag("posts")
  updateTag(`post-${post.id}`)
}
```

Rules:

Use `updateTag` when:

* user immediately expects to see the change
* implementing **read-your-own-writes**

Example:

```
createPost()
redirect("/posts")
```

The new post appears instantly.

---

# 8. Revalidating Routes With revalidatePath

`revalidatePath` refreshes a specific route.

Example:

```tsx
import { revalidatePath } from "next/cache"

export async function updateProfile() {
  await db.user.update()

  revalidatePath("/profile")
}
```

Use when:

* page depends on multiple data sources
* route must refresh entirely

Prefer tag-based invalidation when possible.

---

# 9. Server Actions and Cache Invalidation

Mutations should follow this pattern:

```
Server Action
   ↓
mutate database
   ↓
updateTag OR revalidateTag
   ↓
redirect or return
```

Example:

```tsx
"use server"

export async function createPost(data: FormData) {
  await db.post.create()

  updateTag("posts")
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mark0025/npm-auth-gateway](https://github.com/Mark0025/npm-auth-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
