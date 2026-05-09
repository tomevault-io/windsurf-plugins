---
trigger: always_on
description: This is a Raycast extension for browsing DTF.ru (Russian gaming and entertainment portal). It uses the public DTF API (Osnova platform) to fetch posts, news, and search content.
---

# DTF Raycast Extension - Development Guidelines

## Overview

This is a Raycast extension for browsing DTF.ru (Russian gaming and entertainment portal). It uses the public DTF API (Osnova platform) to fetch posts, news, and search content.

## Project Structure

```text
src/
├── api/
│   ├── client.ts      # HTTP client, API methods, caching
│   └── types.ts       # TypeScript types for API responses
├── components/
│   ├── AIPostView.tsx       # AI-powered post analysis view
│   ├── PeriodDropdown.tsx   # Time period selector for Popular
│   ├── PostActions.tsx      # Action panel for posts
│   ├── PostDetail.tsx       # List item detail preview
│   ├── PostDetailView.tsx   # Full-screen Detail view with rich content
│   ├── PostList.tsx         # Reusable post list component
│   ├── SortingDropdown.tsx  # Sorting selector for Latest
│   ├── SubsiteSortingDropdown.tsx # Sorting for Topics/Blogs
│   └── index.ts             # Component exports
├── utils/
│   ├── ai-helpers.ts  # AI prompts and text extraction
│   └── formatters.ts  # Date/number formatting utilities
├── tools/             # AI tools for @dtf mentions
│   ├── get-news.ts
│   ├── get-popular-posts.ts
│   ├── get-topic-posts.ts
│   ├── get-topics.ts
│   └── search-posts.ts
├── latest-posts.tsx   # Latest posts command
├── popular-posts.tsx  # Popular posts command
├── news.tsx           # News command
├── search-posts.tsx   # Search command
├── browse-subsites.tsx # Topics browser command
├── top-blogs.tsx      # Top blogs command
└── menu-bar.tsx       # Menu bar command (macOS only)
```

## DTF API Reference

### Base URLs

- Primary: `https://api.dtf.ru/v2.10/` (recommended)
- Legacy: `https://api.dtf.ru/v2.1/`

### Key Endpoints

| Endpoint | Description | Pagination |
|----------|-------------|------------|
| `/v2.10/timeline` | General feed | `cursor` |
| `/v2.10/timeline?subsitesIds=ID` | Posts by subsite | `cursor` |
| `/v2.10/feed?sorting=hotness&pageName=popular` | Popular posts | `cursor` |
| `/v2.10/news` | Editorial news | `lastId`, `lastSortingValue` |
| `/v2.10/search?query=TEXT&order_by=relevant` | Search | N/A |
| `/v2.10/discovery/topics` | Topics list | N/A |
| `/v2.10/discovery/blogs` | Top blogs | N/A |

### Response Structure

```typescript
interface ApiResponse<T> {
  message: string;
  result: T;
  error?: { code: number; info: unknown[] };
}

// Timeline/Feed response
interface TimelineResult {
  items: TimelineItem[];
  cursor?: string;
  lastId?: number;
}

// News response
interface NewsResult {
  news: Post[];
  lastId?: number;
}

// Search response - IMPORTANT: uses "contents", not "items"
interface SearchResult {
  contents: TimelineItem[];
  subsites?: { users: [], subsites: [] };
  lastId?: number;
}
```

### Pagination Notes

1. **Standard pagination**: Use `cursor` parameter (v2.10+)
2. **News pagination**: Requires BOTH `lastId` AND `lastSortingValue`
   - `lastSortingValue` is typically the `date` field of the last post
   - Example: `?lastId=4416613&lastSortingValue=1764226095`

### Image URLs

All images use the Leonardo CDN:

```text
https://leonardo.osnova.io/{UUID}/-/preview/{SIZE}/
```

Common sizes: `100/` (avatars), `400/`, `800/` (covers)

### Personal Blog Detection

When `author.id === subsite.id`, it's a personal blog - don't show subsite separately.

## Raycast Best Practices

### Action Priority

1. `Action.Push` → Opens Detail view (Enter key)
2. `Action.OpenInBrowser` → Opens in browser (Cmd+Enter)

### Platform-Specific Shortcuts

```typescript
shortcut={{
  macOS: { modifiers: ["cmd"], key: "return" },
  Windows: { modifiers: ["ctrl"], key: "return" },
}}
```

### Icons

- Always use circular mask for avatars: `{ source: url, mask: Image.Mask.Circle }`
- Fallback to author avatar if subsite avatar is missing

### Pagination with useFetch

```typescript
const { data, pagination } = useFetch(
  (options) => `${URL}?cursor=${options.cursor || ""}`,
  {
    mapResult(result) {
      return {
        data: transformedPosts,
        hasMore: !!result.result?.cursor && posts.length > 0,
        cursor: result.result?.cursor,
      };
    },
    keepPreviousData: true,
    initialData: [],
  }
);

// Pass pagination to List
<List pagination={pagination}>
```

### Filtering Duplicates

API may return duplicate posts during pagination:

```typescript
const uniquePosts = useMemo(() => {
  const seen = new Set<number>();
  return data.filter((post) => {
    if (seen.has(post.id)) return false;
    seen.add(post.id);
    return true;
  });
}, [data]);
```

### List.Dropdown for Filters

```typescript
<List
  searchBarAccessory={
    <List.Dropdown tooltip="Filter" onChange={setValue}>
      <List.Dropdown.Item title="Option" value="value" />
    </List.Dropdown>
  }
>
```

### Detail Metadata Toggle

Use `showMetadata` prop to allow hiding metadata panel:

```typescript
<PostDetail post={post} showMetadata={showMetadata} />
```

## AI Features

The extension includes AI-powered features:

### Available AI Actions

- **Summarize** (`⌘S` / `Ctrl+S`) - 2-3 paragraph summary
- **Translate** (`⌘T` / `Ctrl+T`) - Translation to English

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadeov/dtf-raycast](https://github.com/shadeov/dtf-raycast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
