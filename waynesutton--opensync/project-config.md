---
trigger: always_on
description: This document covers security patterns specific to the markdown-blog application, a Convex-powered blog with no authentication.
---

# Security Guidelines for markdown-blog

This document covers security patterns specific to the markdown-blog application, a Convex-powered blog with no authentication.

## App-Specific Security Context

### Architecture

- **Frontend**: Vite + React 18.2 SPA (client-side only)
- **Backend**: Convex.dev (serverless database and functions)
- **Hosting**: Netlify with edge functions
- **Auth**: None (public blog)

### React Server Components Vulnerabilities

**Status: NOT AFFECTED**

This app does NOT use React Server Components and is NOT affected by:

- CVE-2025-55182 (Remote Code Execution)
- CVE-2025-55184 (Denial of Service)
- CVE-2025-55183 (Source Code Exposure)

These vulnerabilities affect apps using:

- `react-server-dom-webpack`
- `react-server-dom-parcel`
- `react-server-dom-turbopack`

This app uses standard React 18.2.0 client-side rendering with Vite bundler.

For the latest information, see:

- https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components
- https://react.dev/blog/2025/12/11/denial-of-service-and-source-code-exposure-in-react-server-components

## Database Tables

| Table        | Contains PII | Public Access | Notes                              |
| ------------ | ------------ | ------------- | ---------------------------------- |
| `posts`      | No           | Read-only     | Blog content                       |
| `viewCounts` | No           | Write via API | View counter per post              |
| `siteConfig` | No           | Internal      | Site settings (not currently used) |

## 1. Public API Security

### Query Functions (Read-Only)

All queries in this app are intentionally public for blog content:

```typescript
// Public queries - safe for public access
export const getAllPosts = query({...});      // List published posts
export const getPostBySlug = query({...});    // Get single post
export const getViewCount = query({...});     // Get view count
```

### Mutation Functions

| Function             | Risk Level | Notes                            |
| -------------------- | ---------- | -------------------------------- |
| `syncPostsPublic`    | Medium     | Build-time sync, no auth         |
| `incrementViewCount` | Low        | No rate limiting, but low impact |

### syncPostsPublic Security Consideration

The `syncPostsPublic` mutation allows syncing posts without authentication. This is intentional for build-time deployment but has security implications:

```typescript
// Current: No auth check
export const syncPostsPublic = mutation({
  args: { posts: v.array(...) },
  handler: async (ctx, args) => {
    // Syncs posts directly
  },
});
```

**Mitigations in place:**

1. Mutation only affects the `posts` table
2. Posts require specific schema (slug, title, content, etc.)
3. Build-time sync uses environment variables

**Recommendations:**

- Consider adding CONVEX_DEPLOY_KEY check for production
- Monitor for unusual sync activity in Convex dashboard

## 2. HTTP Endpoint Security

### XSS Prevention

All HTTP endpoints properly escape output:

```typescript
// HTML escaping for Open Graph
function escapeHtml(text: string): string {
  return text
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
}

// XML escaping for RSS feeds
function escapeXml(text: string): string {
  return text
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&apos;");
}
```

### CORS Headers

API endpoints include CORS headers for public access:

```typescript
headers: {
  "Content-Type": "application/json; charset=utf-8",
  "Cache-Control": "public, max-age=300, s-maxage=600",
  "Access-Control-Allow-Origin": "*",
}
```

This is intentional for a public blog API.

### HTTP Endpoints

| Route           | Method | Auth | Description                      |
| --------------- | ------ | ---- | -------------------------------- |
| `/rss.xml`      | GET    | No   | RSS feed (descriptions)          |
| `/rss-full.xml` | GET    | No   | Full RSS feed (content for LLMs) |
| `/sitemap.xml`  | GET    | No   | XML sitemap for SEO              |
| `/api/posts`    | GET    | No   | JSON post list                   |
| `/api/post`     | GET    | No   | Single post JSON/markdown        |
| `/meta/post`    | GET    | No   | Open Graph HTML for crawlers     |

## 3. Edge Function Security

### Bot Detection (botMeta.ts)

The edge function detects social media crawlers and serves Open Graph metadata:

```typescript
// Bot user agent detection
const BOTS = [
  "facebookexternalhit",
  "twitterbot",
  // ... more bots
];
```

**Security considerations:**

- User agent can be spoofed, but this only affects OG metadata delivery
- Fallback to SPA for non-bots is secure
- No sensitive data exposed to bots

## 4. Client-Side Security

### Markdown Rendering

Uses `react-markdown` with controlled components:

- External links open with `rel="noopener noreferrer"`
- Images use lazy loading
- No raw HTML injection (markdown only)

### Copy to Clipboard


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/opensync](https://github.com/waynesutton/opensync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
