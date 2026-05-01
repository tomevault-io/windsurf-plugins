---
trigger: always_on
description: Instructions for AI coding agents working on this codebase.
---

# AGENTS.md

Instructions for AI coding agents working on this codebase.

## Project overview

Your content is instantly available to browsers, LLMs, and AI agents.. Write markdown, sync from the terminal. Your content is instantly available to browsers, LLMs, and AI agents. Built on Convex and Netlify.

## Default and legacy modes

- Default auth mode: `convex-auth`
- Default hosting mode: `convex-self-hosted`
- Legacy compatibility mode: `workos` auth + `netlify` hosting

**Key features:**
- Markdown posts with frontmatter
- Four themes (dark, light, tan, cloud)
- Full text search with Command+K
- Real-time analytics at `/stats`
- RSS feeds and sitemap for SEO
- API endpoints for AI/LLM access

## Current Status

- **Site Name**: markdown sync
- **Site Title**: markdown sync framework
- **Site URL**: https://yoursite.example.com
- **Total Posts**: 21
- **Total Pages**: 4
- **Latest Post**: 2026-03-20
- **Last Updated**: 2026-03-21T00:12:49.188Z

## Tech stack

| Layer | Technology |
|-------|------------|
| Frontend | React 18, TypeScript, Vite |
| Backend | Convex (real-time serverless database) |
| Styling | CSS variables, no preprocessor |
| Hosting | Convex self-hosting (default) or Netlify (legacy) |
| Auth | @robelest/convex-auth (default) or WorkOS (legacy) |
| Content | Markdown with gray-matter frontmatter |

## Setup commands

```bash
npm install                    # Install dependencies
npx convex dev                 # Initialize Convex (creates .env.local)
npm run dev                    # Start dev server at http://localhost:5173
```

## Content sync commands

```bash
npm run sync                   # Sync markdown to development Convex
npm run sync:prod              # Sync markdown to production Convex
npm run import <url>           # Import external URL as markdown post
```

Content syncs instantly. No rebuild needed for markdown changes.

## Build and deploy

```bash
npm run build                  # Build for production
npx convex deploy              # Deploy Convex functions to production
npm run deploy                 # Deploy with Convex self-hosting
```

**Netlify build command:**
```bash
npm ci --include=dev && npx convex deploy --cmd 'npm run build'
```

## Code style guidelines

- Use TypeScript strict mode
- Prefer functional components with hooks
- Use Convex validators for all function arguments and returns
- Always return `v.null()` when functions don't return values
- Use CSS variables for theming (no hardcoded colors)
- No emoji in UI or documentation
- No em dashes between words
- Sentence case for headings

## Convex patterns (read this)

### Always use validators

Every Convex function needs argument and return validators:

```typescript
export const myQuery = query({
  args: { slug: v.string() },
  returns: v.union(v.object({...}), v.null()),
  handler: async (ctx, args) => {
    // ...
  },
});
```

### Always use indexes

Never use `.filter()` on queries. Define indexes in schema and use `.withIndex()`:

```typescript
// Good
const post = await ctx.db
  .query("posts")
  .withIndex("by_slug", (q) => q.eq("slug", args.slug))
  .first();

// Bad - causes table scans
const post = await ctx.db
  .query("posts")
  .filter((q) => q.eq(q.field("slug"), args.slug))
  .first();
```

### Make mutations idempotent

Mutations should be safe to call multiple times:

```typescript
export const heartbeat = mutation({
  args: { sessionId: v.string(), currentPath: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    const now = Date.now();
    const existing = await ctx.db
      .query("activeSessions")
      .withIndex("by_sessionId", (q) => q.eq("sessionId", args.sessionId))
      .first();

    if (existing) {
      // Early return if recently updated with same data
      if (existing.currentPath === args.currentPath && 
          now - existing.lastSeen < 10000) {
        return null;
      }
      await ctx.db.patch(existing._id, { currentPath: args.currentPath, lastSeen: now });
      return null;
    }

    await ctx.db.insert("activeSessions", { ...args, lastSeen: now });
    return null;
  },
});
```

### Patch directly without reading

When you only need to update fields, patch directly:

```typescript
// Good - patch directly
await ctx.db.patch(args.id, { content: args.content });

// Bad - unnecessary read creates conflict window
const doc = await ctx.db.get(args.id);
if (!doc) throw new Error("Not found");
await ctx.db.patch(args.id, { content: args.content });
```

### Use event records for counters

Never increment counters on documents. Use separate event records:

```typescript
// Good - insert event record
await ctx.db.insert("pageViews", { path, sessionId, timestamp: Date.now() });

// Bad - counter updates cause write conflicts
await ctx.db.patch(pageId, { views: page.views + 1 });
```

### Frontend debouncing

Debounce rapid mutations from the frontend. Use refs to prevent duplicate calls:

```typescript
const isHeartbeatPending = useRef(false);
const lastHeartbeatTime = useRef(0);

const sendHeartbeat = useCallback(async (path: string) => {
  if (isHeartbeatPending.current) return;
  if (Date.now() - lastHeartbeatTime.current < 5000) return;
  
  isHeartbeatPending.current = true;
  lastHeartbeatTime.current = Date.now();
  
  try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/markdown-site](https://github.com/waynesutton/markdown-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
