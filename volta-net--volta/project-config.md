---
trigger: always_on
description: Volta is a **GitHub inbox replacement** — a fast, beautiful way to manage GitHub notifications, issues, and pull requests. Think of it as the inbox GitHub should have built.
---

# Volta

Volta is a **GitHub inbox replacement** — a fast, beautiful way to manage GitHub notifications, issues, and pull requests. Think of it as the inbox GitHub should have built.

## Vision

- Replace GitHub's notification inbox with something actually usable
- Unified view of issues, PRs, releases, and CI status across all your repos
- Real-time sync via GitHub App webhooks
- Beautiful, keyboard-first UI for power users

## Tech Stack

- **Framework**: Nuxt 4 with Vue 3 Composition API (`ssr: false` - client-only SPA)
- **UI**: Nuxt UI v4 (bleeding edge) + Tailwind CSS
- **Auth**: nuxt-auth-utils with GitHub App OAuth
- **GitHub API**: Octokit with @octokit/auth-app
- **Database**: NuxtHub with Drizzle ORM (PostgreSQL)
- **Editor**: TipTap with custom extensions
- **AI**: Vercel AI SDK (`ai` package) for issue completion
- **Durable Functions**: Vercel Workflow DevKit for long-running operations

## Critical Context

### SSR is Disabled
This app runs as a client-only SPA. This means:
- `useFetch` still works but runs client-side only
- No server-side rendering, no hydration issues
- Auth loads client-only (`auth.loadStrategy: 'client-only'`)

### Nuxt UI v4
We use Nuxt UI v4 (not v3). Key differences:
- Components use `slots` API in `app.config.ts` for customization
- Theme colors defined in `ui.theme.colors` array
- Use `UDashboardPanel`, `UDashboardSidebar`, `UDashboardNavbar` for layout
- Use `UEditor` for rich text editing

## Architecture

### GitHub App Integration

- OAuth flow for user authentication (`/auth/github`)
- Webhook endpoint (`/webhook`) receives all GitHub events
- Installation API (`/api/installations`) lists repos the user has access to
- Sync API (`/api/repositories/[owner]/[name]/sync`) triggers full repo sync
- Server utilities in `server/utils/octokit.ts` for GitHub API calls

### Webhook Events

We subscribe to these GitHub events to keep Volta in sync:
- `issues`, `issue_comment` - Task management
- `pull_request`, `pull_request_review`, `pull_request_review_comment`, `pull_request_review_thread` - PR tracking
- `label`, `milestone` - Organization (tags, cycles)
- `repository` - Repo changes (archived, renamed, etc.)
- `push`, `release` - Activity feed
- `installation`, `installation_repositories`, `installation_target`, `meta` - App lifecycle

### Key Directories

- `app/pages/` - Vue pages (index, inbox, settings)
- `app/components/` - Reusable UI components
- `app/composables/` - Client-side composables (`use*.ts`)
- `server/api/` - API routes (notifications, installations, sync)
- `server/db/` - Database schema and migrations (Drizzle ORM)
- `server/routes/` - Auth and webhook handlers
- `server/utils/` - Server utilities (Octokit, webhooks, sync)
- `server/workflows/` - Durable workflows (Vercel Workflow DevKit)
- `shared/types/` - Shared TypeScript types

### Durable Workflows

We use [Vercel Workflow DevKit](https://useworkflow.dev) for long-running operations that would timeout in serverless environments. Workflows use the `"use workflow"` and `"use step"` directives.

**Key workflows:**
- `syncRepository.ts` - Full repository sync (collaborators, labels, milestones, issues)

Note: Webhooks and individual issue sync run synchronously (not as workflows) since they need immediate responses.

**Workflow pattern:**
```ts
// server/workflows/example.ts
"use workflow";

export async function myWorkflow(input: Input) {
  // Each step is durable and resumable
  const result1 = await (async () => {
    "use step";
    return await longRunningOperation();
  })();

  const result2 = await (async () => {
    "use step";
    return await anotherOperation(result1);
  })();

  return { result1, result2 };
}
```

**Triggering workflows:**
```ts
import { start } from 'workflow'
import { myWorkflow } from '~/server/workflows/example'

// In an API route
await start(myWorkflow, [{ input: 'value' }])
```

## API Route Conventions

Server routes use file-based routing with HTTP verb suffixes:

```
server/api/
  notifications.get.ts        → GET /api/notifications
  notifications/
    [id].patch.ts             → PATCH /api/notifications/:id
    [id].delete.ts            → DELETE /api/notifications/:id
    read-all.post.ts          → POST /api/notifications/read-all
  repositories/
    [owner]/[name]/
      issues/[number]/
        index.get.ts          → GET /api/repositories/:owner/:name/issues/:number
        body.patch.ts         → PATCH /api/repositories/:owner/:name/issues/:number/body
```

## Database

The database uses auto-generated `id` (serial) as primary key, with `githubId` for GitHub lookups.

**ID Strategy:**
- `id` - Auto-generated stable ID (primary key)
- `githubId` - GitHub's ID (for API lookups, indexed)
- Unique constraint on `(repositoryId, number)` for issues

**Key Tables:** users, installations, repositories, issues, labels, milestones, notifications

**Drizzle Workflow:**
1. Modify schema in `server/db/schema.ts`
2. Run `pnpm db:generate` to generate migration
3. Run `pnpm dev` to apply migration during development

**Database Access:**

```ts
import { eq } from 'drizzle-orm'
import { db, schema } from '@nuxthub/db'

// Find by GitHub ID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [volta-net/volta](https://github.com/volta-net/volta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
